# Salida de la variante sin restricciones

Lo corrimos con `ollama run qwen-demo --think=false --nowordwrap < reto1\prompt_variante.txt` y la copiamos tal cual salió, el corte del final es del modelo, no nuestro

```python
from datetime import datetime, timedelta
from typing import Dict, Set, Optional

def registrar_entrada(
    vehiculo: dict, 
    tipo_usuario: str, 
    ocupados: Dict[str, int], 
    carnes_dentro: Set[str], 
    hora_entrada: datetime
) -> Dict:
    """
    Decide si un vehículo puede ingresar al parqueadero ParqueFET y registra su entrada si es posible.

    Args:
        vehiculo (dict): Objeto con claves 'placa' (str), 'tipo' ('carro' o 'moto'), 
                         'carne_id' (str) y 'carne_vigente' (bool).
        tipo_usuario (str): Tipo de usuario, puede ser "estudiante", "personal" o "visitante".
        ocupados (dict): Diccionario con claves "carros" e "motos" indicando cantidad actual.
        carnes_dentro (set): Conjunto con los carne_id que ya tienen un vehículo dentro.
        hora_entrada (datetime): Hora y fecha de la entrada solicitada.

    Returns:
        dict: {
            "permitido": bool, 
            "motivo": str, 
            "tarifa_hora": int, 
            "hora_limite_salida": datetime | None
        }
    """
    
    # Regla 1: Solo ingresan vehículos registrados a un carné vigente.
    if not vehiculo['carne_vigente']:
        return {
            "permitido": False,
            "motivo": "El vehículo no tiene un carné vigente.",
            "tarifa_hora": 0,
            "hora_limite_salida": None
        }

    # Regla 4: Un carné solo puede tener un vehículo dentro a la vez.
    if vehiculo['carne_id'] in carnes_dentro:
        return {
            "permitido": False,
            "motivo": f"El carné {vehiculo['carne_id']} ya tiene un vehículo dentro.",
            "tarifa_hora": 0,
            "hora_limite_salida": None
        }

    # Regla 2: Capacidad: 120 carros y 300 motos.
    tipo_vehiculo = vehiculo['tipo']
    capacidad_maxima = {"carros": 120, "motos": 300}
    
    if ocupados[tipo_vehiculo] >= capacidad_maxima[tipo_vehiculo]:
        return {
            "permitido": False,
            "motivo": f"La capacidad de {tipo_vehiculo} está llena.",
            "tarifa_hora": 0,
            "hora_limite_salida": None
        }

    # Regla 5: Ningún vehículo puede permanecer más de 14 horas seguidas.
    # Se asume que el tiempo máximo se calcula desde la última entrada registrada 
    # para ese carné específico (simulado aquí como si fuera la última vez que entró).
    # Nota: En un sistema real, esto requeriría consultar el historial del carné.
    # Para este ejercicio, asumimos una
