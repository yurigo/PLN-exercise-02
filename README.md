# Sallexa v2.0 - Asistente Médico con Memoria y Lógica

## Contexto: El problema de la v1.0

En la sesión anterior construimos Sallexa v1.0, un sistema capaz de clasificar frases sueltas (Síntoma, Urgencia, Administrativo, Ruido). Sin embargo, nuestro cliente ha detectado un problema grave: El sistema tiene amnesia.

Ejemplo del error:

```
Usuario: "Tengo fiebre".
Sallexa v1: Detecta SINTOMA. (Correcto)
Usuario: "39 grados".
Sallexa v1: Detecta RUIDO o ADMINISTRATIVO (Error). No sabe que "39 grados" está conectado con la frase anterior.
```

Misión de hoy: Dotar al sistema de Memoria (Contexto), Entendimiento (Extracción de Entidades) y Criterio (Lógica y Ética).

## Paso 1: Gestión del Diálogo (Máquina de Estados)

Para solucionar la "amnesia", dejaremos de procesar frases aisladas. Implementaremos una **Máquina de Estados Finitos (FSM)**. El bot debe saber en qué punto de la conversación se encuentra.

Tareas:

1. Define una variable o clase que controle el `estado_actual`.
2. Implementa la siguiente lógica de flujo básica:

- Estado 0 (IDLE): Esperando al usuario. Si detecta intención SINTOMA pasa a Estado 1.
- Estado 1 (RECABANDO_DATOS): El bot pregunta "¿Desde cuándo?" o "¿Qué gravedad?". Lo que el usuario responda aquí se guarda asociado al síntoma. (ver paso 2).
- Estado 2 (URGENCIA): Si se detecta peligro, se entra en bucle de protocolo de emergencia hasta que el usuario confirme que ha llamado al 112. (ver paso 3).
- Estado 3 (PROPORCIONANDO_RECOMENDACIONES): Si no es urgente, el bot da recomendaciones basadas en los datos recopilados. (ver paso 3).
- Estado 4 (FINALIZAR): El bot despide al usuario y resetea el estado a IDLE.

## Paso 2: Los Oídos (Extracción de Entidades con NLP y Slot filling)

Ya sabemos que el usuario tiene un síntoma, pero necesitamos extraer el dato concreto del texto natural.

Input: "Me duele la cabeza desde hace 3 días"

- Sallexa v1: Decía "Es un síntoma".
- Sallexa v2: Debe extraer: { "sintoma": "dolor de cabeza", "duracion": "3 días" }.

El sistema experto (cerebro) veremos que necesitará datos estructurados para razonar. Por ejemplo:

```python
contexto_paciente = {
    "intencion_actual": None,  # Ej: SINTOMA
    "slots": {
        "tipo_sintoma": None,  # Ej: Dolor de cabeza
        "duracion": None,      # Ej: 3 días
        "temperatura": None,   # Ej: 38.5
        "gravedad_percibida": None
        [etc.]
    }
}
```

Deberíamos implementar una técnica llamada **Slot Filling (rellenado de huecos)** para extraer y almacenar estos datos en el contexto del paciente.

Ejemplo de contexto actualizado tras extracción.

```python
contexto_paciente = {
    "intencion_actual": "SINTOMA",  # Ej: SINTOMA
    "slots": {
        "tipo_sintoma": "dolor de cabeza",
        "duracion": "3 dias",
        "temperatura": None,
        "gravedad_percibida": None,
        [etc.]
    }
}
```

Tareas:

Mejora tu pipeline de NLP (usando spaCy o re - Expresiones Regulares).

Crea extractores para:

- Síntomas: Detectar patrones como "dolor de cabeza", "fiebre", "tos".
- Duración: Detectar patrones como "desde ayer", "3 días", "una semana".
- Temperaturas: Detectar patrones como "38.5", "39 grados", "40º".
- Gravedad: Detectar adjetivos como "leve", "moderado", "grave".
- etc

Actualiza los slots del contexto dinámicamente. Cuando el contexto tenga los datos mínimos necesarios, el sistema debe pasar a la fase de decisión.

## Paso 3: El Cerebro (Sistema Experto)

Un sistema experto necesita datos estructurados para razonar.

Estructura de Memoria (Contexto): El bot debe manejar un diccionario o objeto JSON interno por paciente similar a esto:

```python
contexto_paciente = {
    "intencion_actual": "SINTOMA",  # Ej: SINTOMA
    "slots": {
        "tipo_sintoma": "dolor de cabeza",
        "duracion": "3 dias",
        "temperatura": None,
        "gravedad_percibida": None
    }
}
```

A continuación, implementa el cerebro lógico con logica de motor de inferencia basado en reglas IF-THEN ó fuzzy logic.

### Lógica del Motor de Inferencia (Reglas IF-THEN)

Una vez rellenados los slots, el sistema debe decidir. Por ejemplo:

- `SI (sintoma == 'fiebre' Y temperatura > 39) ENTONCES respuesta = 'URGENCIA_ALTA'`

- `SI (sintoma == 'dolor' Y zona == 'pecho') ENTONCES respuesta = 'PROTOCOLO_INFARTO'`

- `SI (sintoma == 'tos' Y duracion > '7 dias') ENTONCES respuesta = 'CITA_PREVIA'`

- `etc.`

### Fuzzy Logic

Alternativamente, puedes implementar lógica difusa para manejar incertidumbre en los síntomas y diagnósticos.

- Define conjuntos difusos para síntomas y urgencias (por ejemplo, "leve", "moderado", "grave").

- Implementa reglas difusas para evaluar la gravedad basada en múltiples factores (por ejemplo, duración del síntoma, temperatura, etc.).

- Calcula el grado de pertenencia a cada conjunto y toma decisiones basadas en estos valores.

## Paso 4: Crash Test

Prueba tu sistema con los siguientes diálogos y verifica que el contexto se mantiene correctamente:

Diálogo 1:
```
Usuario: "Tengo fiebre"
Sallexa: "¿Desde cuándo tienes fiebre?"
Usuario: "Desde hace 2 días"
Sallexa: "¿Cuál es tu temperatura?"
Usuario: "39 grados"
Sallexa: "Esto es una URGENCIA_ALTA. Por favor, llama al 112."

Diálogo 2:
Usuario: "Me duele el pecho"
Sallexa: "¿Desde cuándo tienes dolor en el pecho?"
Usuario: "Desde esta mañana"
Sallexa: "Esto podría ser un PROTOCOLO_INFARTO. Por favor, llama al 112."

Diálogo 3:
Usuario: "Tengo tos"
Sallexa: "¿Desde cuándo tienes tos?"
Usuario: "Una semana"
Sallexa: "Te recomiendo pedir una CITA_PREVIA con tu médico."
```

Implementa estos diálogos y asegúrate de que el sistema responde correctamente, manteniendo el contexto y extrayendo las entidades necesarias.

```
Dialogo ambiguo:
"Me duele el pie muchísimo y creo que me estoy muriendo"
Usuario- ¿Tu bot lo clasifica como dolor leve (pie) o urgencia vital (muriendo)?
```

## Paso 5: Tareas Éticas y de Seguridad:

- Disclaimer Legal: El bot debe iniciar la conversación informando que es un asistente automático y no un médico (Transparencia).
- Manejo de Errores (Safety): Si el bot no entiende o tiene una confianza baja (< 0.6) en su predicción, debe derivar a un humano inmediatamente ("No estoy seguro, te paso con un enfermero").
- Reflexión (Para entregar en el README): Explica cómo tu código intenta mitigarlos (o qué le falta para hacerlo):
- Privacidad: ¿Dónde se guardan los datos del paciente? (GDPR).
- Sesgo: ¿Entendería el bot expresiones de dolor de diferentes culturas o edades?
- Responsabilidad: Si el bot dice "tómate un ibuprofeno" y el paciente es alérgico, ¿de quién es la culpa?
