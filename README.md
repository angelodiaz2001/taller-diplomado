# Taller · Ingeniería de prompts y de contexto Grupo 6

**Integrantes:** Angelo David Diaz, Elian Danilo Romero, Andrés Ricardo Rodríguez

**Caso:** 1 · ParqueFET (control del parqueadero del campus)

**Modelo:** `qwen-demo` (qwen3.5:4b, temperature 0, seed 42, num_predict 700) con Ollama 0.40.7

**Equipo:** Windows 10 Pro, Visual Studio Code y la terminal cmd

## Cómo se ejecutan los prompts

Desde la raíz del repo, en una terminal **cmd** de VS Code:

```bat
ollama run qwen-demo --think=false --nowordwrap < reto1\prompt_malo.txt
ollama run qwen-demo --think=false --nowordwrap < reto1\prompt_formula.txt
ollama run qwen-demo --think=false --nowordwrap < reto1\prompt_variante.txt
```

Le pusimos `--nowordwrap` para que la salida que copiamos en los `.md` no quedara con cortes de línea raros de la terminal

## Reto 1 · Rescatar un prompt malo

Todo está en [`reto1/`](reto1/), ahí están los tres prompts, sus salidas, la [comparación](reto1/comparacion.md) y las capturas de la terminal en [`reto1/evidencias/`](reto1/evidencias/)

## Reflexión

Lo que más pesó fue meter las reglas de negocio en `<reglas>`, con el prompt de una línea el modelo hizo una función en JavaScript que no tenía nada que ver con el parqueadero, y apenas le dimos las reglas ya cumplía casi todas. Lo curioso es que el error que quedó, el cupo que nunca se llena, salió de nuestro propio prompt por poner "carro" en un lado y "carros" en otro, así que el modelo no es el único que se equivoca

Si mañana le pidiéramos a una IA que programe algo del parqueadero no le soltaríamos una frase y ya, le daríamos las reglas, los datos exactos que recibe la función y qué debe devolver, revisaríamos que el prompt no se contradiga y probaríamos el código antes de confiar en él
