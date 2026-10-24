# Reto 1, rescatar un prompt malo

Caso 1 ParqueFET, todo lo corrimos con `qwen-demo` (qwen3.5:4b con temperatura 0 y semilla 42) y `--think=false`, así que si lo vuelven a correr les sale lo mismo

En la carpeta están el prompt malo (`prompt_malo.txt`), el que armamos con la fórmula (`prompt_formula.txt`) y la variante (`prompt_variante.txt`), que es el mismo prompt nuevo pero sin el bloque de restricciones, cada uno con su salida en `.md` y las capturas de la terminal en `evidencias/`

## Lo que el modelo tuvo que adivinar

Con "Hazme una función para registrar la entrada de un vehículo" el modelo tuvo que inventarse casi todo:

1. El lenguaje, se fue por JavaScript diciendo que es "el estándar para la lógica del navegador", y nosotros trabajamos en Python
2. Los datos del vehículo, se inventó marca, modelo, color y conductor, pero nada del carné ni de si es carro o moto
3. Las reglas, no aplicó ninguna, ni carné vigente, ni cupo, ni un vehículo por carné, ni las 14 horas
4. Qué devuelve, un objeto con un ID armado con la fecha, pero nunca dice si el carro puede entrar o no
5. Cómo maneja los errores, lanza excepciones y hace `console.log`, cuando en portería lo que sirve es un motivo claro
6. Dónde se guarda, sugiere una base de datos o localStorage, o sea en el navegador
7. La tarifa, ni la menciona, no sabe que hay visitantes que pagan
8. Cuánto escribir, metió un ejemplo de uso y una lista de características, se gastó los 700 tokens y quedó cortado en "Guarda automáticamente la"

O sea, la función no está mal hecha, solo que es para otro sistema, y eso es culpa del prompt porque no le dimos ninguna regla

## Cómo lo reescribimos

Usamos los cinco bloques de la fórmula:

| Bloque | Qué le pusimos |
|---|---|
| Qué quiero | Una función en Python 3.12 llamada `registrar_entrada` que decida si el vehículo entra y devuelva el registro |
| Para quién | Programadores junior de la FET, código fácil de leer y de probar con pytest |
| Contexto | Qué es ParqueFET y los parámetros exactos que recibe la función |
| Formato de salida | Un solo bloque de código, un diccionario de retorno fijo y una lista de qué regla cubre cada parte |
| Restricciones | Solo librería estándar, no modificar los argumentos, lleno es igual a la capacidad, no inventar reglas y máximo 40 líneas |

Las cinco reglas del parqueadero las metimos entre `<reglas>` y `</reglas>`, aclarando que son datos del sistema, para que no se mezclen con lo que le estamos pidiendo

## Comparación regla por regla

| Regla | Prompt malo | Con fórmula | Variante sin restricciones |
|---|---|---|---|
| Python y el nombre que pedimos | No, salió en JavaScript | Sí | Sí |
| 1, carné vigente | No | Sí | Sí |
| 2, capacidad de 120 carros y 300 motos | No | A medias, la escribe pero nunca bloquea | A medias, se rompe con `KeyError` |
| 3, tarifa de visitantes | No | Sí, 3.000 el carro y 1.500 la moto | No alcanzó a escribirla |
| 4, un vehículo por carné | No | Sí | Sí |
| 5, máximo 14 horas | No | Calcula bien la hora límite, pero se inventa un campo | Solo comentarios, sin código |
| Devuelve el diccionario pedido | No | Sí | No, quedó sin el `return` final |
| Lista de reglas al final | No | No, se cortó antes | No, se cortó antes |

Pasamos de cero reglas a cuatro bien hechas, y la quinta por lo menos ya aparece, cambió el lenguaje, los parámetros y lo que devuelve porque ya no tuvo que adivinar nada de eso

## Lo que sigue fallando

**El cupo nunca se llena,** este es el error más feo:

```python
cap_carros = 120 if tipo == "carro" else 300
ocupados_tipo = ocupados.get(tipo, 0)
if ocupados_tipo >= cap_carros:
```

El tipo del vehículo llega como `"carro"` o `"moto"`, pero en `ocupados` las claves son `"carros"` y `"motos"`, entonces `ocupados.get("carro", 0)` siempre da 0 y el parqueadero nunca se llena. Lo probamos con 120 carros adentro y la función igual dejaba entrar otro, en cambio con una moto de visitante sí cobró bien los 1.500

Lo chistoso es que el error lo causamos nosotros en el prompt, pusimos singular en un lado y plural en el otro y el modelo no se dio cuenta. En la variante pasó lo mismo pero ahí usa `ocupados[tipo]`, así que en vez de fallar callado se cae con `KeyError`. Para arreglarlo toca usar las mismas claves en todo el prompt, por ejemplo `{"carro": int, "moto": int}`

**Se inventa un dato,** para la regla de las 14 horas usa `vehiculo["hora_entrada_anterior"]`, que no existe en lo que le dimos, aunque le dijimos que no inventara. Lo que pasa es que esa regla en realidad se revisa a la salida y no a la entrada, y el modelo igual quiso meterla

**No cumplió todo el formato,** el código tiene como 50 líneas y le pedimos 40, tomó el vehículo como diccionario (`vehiculo["tipo"]`) y no como objeto (`vehiculo.tipo`), aunque ahí nuestro prompt también era ambiguo, y la lista de reglas del final nunca salió porque `qwen-demo` tiene un límite de 700 tokens y se cortó antes

## La variante sin restricciones

Le quitamos solo el bloque de restricciones y dejamos todo lo demás igual:

1. Se puso mucho más largo, docstring con `Args` y `Returns` y cada `return` en varias líneas, se gastó los 700 tokens y no alcanzó a poner la tarifa ni el `return` final, así que el código no sirve
2. Empezó a suponer cosas, en la regla 5 escribe "Se asume que..." y "En un sistema real, esto requeriría consultar el historial...", sin el "no inventes reglas" se pone a especular
3. Lo que no cambió fue el `>=` del cupo, aunque quitamos la frase de que lleno es igual a la capacidad, ahí el modelo ya lo hacía bien solo

Con un modelo local pequeño las restricciones sirven sobre todo para controlar qué tanto escribe y que no se salga del tema, sin ellas se alarga, especula y se corta

## Qué aprendimos

Un prompt de una línea no da una función mala, da una función para otra cosa, las reglas del negocio hay que dárselas porque el modelo no las conoce. Las etiquetas `<reglas>` ayudaron bastante porque el modelo terminó numerando cada regla en los comentarios y así fue fácil revisarlo, igual la fórmula no quita todos los errores, los pasa a los detalles de nuestro propio prompt, por eso toca probar el código y no solo leerlo

Si lo volviéramos a hacer usaríamos las mismas claves para carro y moto en todo lado, diríamos "objeto con atributos", sacaríamos la regla de las 14 horas de la entrada y pediríamos una respuesta más corta para que quepa en los 700 tokens
