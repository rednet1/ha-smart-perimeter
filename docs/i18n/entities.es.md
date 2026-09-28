[English](../entities.md) | [Italiano](entities.it.md) | [Español](entities.es.md) | [日本語](entities.ja.md)

# Hoja de configuración

Rellena esto con tus propias entidades antes de importar los blueprints — es más cómodo tenerlas todas en un solo lugar que buscarlas una a una mientras haces clic en el formulario de entrada de cada blueprint.

| Rol | Dominio de ejemplo | Tu entidad |
|---|---|---|
| Helper de estado armado | `input_boolean.*` | |
| Indicador de modo parcial | `input_boolean.*` o `automation.*` de repuesto | |
| Sirena | `switch.*` | |
| LED / indicador visual (opcional) | `light.*` | |
| Sensor de ID de autenticación (huella/teclado/NFC) | `sensor.*` | |
| Sensores de movimiento | `binary_sensor.*` (device_class: motion) | |
| Sensores de puerta/ventana | `binary_sensor.*` (device_class: door/window/opening) | |
| Sensores de ocupación de mascota (por cámara) | `binary_sensor.*` | |
| Rastreadores de presencia (para auto-armado) | `person.*` / `device_tracker.*` | |
| Destinos de anuncios de voz (opcional) | `assist_satellite.*` | |

Tabla por persona (una fila → una instancia de la automatización `person_trigger.yaml`):

| Persona | ID en el sensor | Notas |
|---|---|---|
| | | |
| | | |
