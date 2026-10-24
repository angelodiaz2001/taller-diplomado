# Salida del prompt con la fórmula

Lo corrimos con `ollama run qwen-demo --think=false --nowordwrap < reto1\prompt_formula.txt` y la copiamos tal cual salió, el corte del final es del modelo, no nuestro

```python
from datetime import datetime, timedelta
from typing import Any

def registrar_entrada(vehiculo: dict, tipo_usuario: str, ocupados: dict, carnes_dentro: set, hora_entrada: datetime) -> dict:
    """
    Decide si un vehículo puede ingresar al ParqueFET según las reglas de negocio.
    
    Reglas verificadas:
    1 (Solo carnés vigentes), 2 (Capacidad), 3 (Tipos/Pagos), 4 (Un carné por vez), 5 (Límite 14h)
    
    Retorna: {"permitido": bool, "motivo": str, "tarifa_hora": int, "hora_limite_salida": datetime | None}
    """
    tipo = vehiculo["tipo"]
    carne_id = vehiculo["carne_id"]
    vigente = vehiculo["carne_vigente"]
    
    # Regla 1: Carné debe ser vigente
    if not vigente:
        return {"permitido": False, "motivo": "Carné no vigente", "tarifa_hora": 0, "hora_limite_salida": None}
    
    # Regla 4: El carné ya tiene un vehículo dentro
    if carne_id in carnes_dentro:
        return {"permitido": False, "motivo": "Carné ya tiene vehículo dentro", "tarifa_hora": 0, "hora_limite_salida": None}
    
    # Regla 2 y 5: Verificar capacidad y tiempo máximo de permanencia
    cap_carros = 120 if tipo == "carro" else 300
    ocupados_tipo = ocupados.get(tipo, 0)
    limite_tiempo = 14 * 60 * 60  # 14 horas en segundos
    
    if ocupados_tipo >= cap_carros:
        return {"permitido": False, "motivo": "Parqueadero lleno", "tarifa_hora": 0, "hora_limite_salida": None}
    
    # Verificar tiempo de permanencia (Regla 5)
    if vehiculo.get("hora_entrada_anterior"):
        duracion = (hora_entrada - vehiculo["hora_entrada_anterior"]).total_seconds()
        if duracion > limite_tiempo:
            return {"permitido": False, "motivo": "Excedió 14 horas de permanencia", "tarifa_hora": 0, "hora_limite_salida": None}
    
    # Regla 3: Calcular tarifa (solo para visitantes)
    tarifa = 0
    if tipo_usuario == "visitante":
        tarifa_carro = 3000
        tarifa_moto = 1500
        tarifa = tarifa_carro if tipo == "carro" else tarifa_moto
    
    # Si todo está bien, registrar entrada
    return {
        "permitido": True, 
        "motivo": "Ingreso autorizado", 
        "tarifa_hora": tarifa, 
        "hora_limite_salida": hora_entrada + timedelta(hours=14)
    }

# Mape
