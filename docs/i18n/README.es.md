[English](../../README.md) | [Italiano](README.it.md) | [Español](README.es.md) | [日本語](README.ja.md)

# Smart Perimeter

Un sistema de alarma/perímetro para Home Assistant construido a partir de blueprints, con dos características que la mayoría de las alarmas caseras no tienen:

1. **Armado/desarmado multipersona** desde cualquier sensor de "quién se autenticó" con ID numérico (lector de huellas dactilares, teclado, lector NFC...), con una única fuente de verdad para el estado armado — así que no importa qué dispositivo o qué persona lo active, el estado nunca se desincroniza.
2. **Modo parcial consciente de mascotas.** Si tienes una mascota que se queda sola en casa, los sensores de movimiento de una alarma normal darán falsas alarmas constantemente por su culpa. Este sistema usa un detector de objetos con IA en cámara (probado con el seguimiento del objeto "dog" de [Frigate](https://frigate.video/), pero funciona con cualquier integración que te dé un sensor binario de "mascota detectada" por cámara) para armarse automáticamente en modo **Completo** (todos los sensores) cuando no hay nadie — ni personas ni mascotas —, o en modo **Parcial** (solo puertas/ventanas, sin movimiento interior) cuando la mascota está sola en casa.

Todo esto viene empaquetado como [Blueprints de Home Assistant](https://www.home-assistant.io/docs/automation/using_blueprints/): los importas, rellenas tus propias entidades, y listo. No hace falta tocar YAML para una configuración básica.

## ¿Por qué no usar simplemente una integración de alarma normal?

Puedes hacerlo, y para lo básico (puerta/sirena) probablemente deberías reutilizar lo que ya te ofrece la integración de tu panel de alarma. Este proyecto existe para cubrir el vacío específico que tienen la mayoría de las alarmas: **no saben distinguir a tu mascota de un intruso**, así que o desactivas permanentemente los sensores de movimiento interiores (seguridad más débil) o convives con falsas alarmas cada vez que la mascota se mueve.

## Arquitectura

```
                    ┌─────────────────────┐
  huella/      ───▶│ Disparador Persona    │
  teclado/NFC       │ (uno por persona)     │
                    └──────────┬───────────┘
                               │ llama a
                               ▼
                    ┌─────────────────────┐        ┌───────────────────────┐
  presencia  ──────▶│   Script Armar       │───────▶│ Script Desarmar       │
  (auto-armado)     │ (decide Completo/     │◀───────│                       │
                    │  Parcial según los    │        └───────────────────────┘
                    │  sensores de cámara)  │
                    └──────────┬───────────┘
                               │ habilita
                               ▼
                    ┌─────────────────────┐        ┌───────────────────────┐
  movimiento/puerta▶│ Disparador Perímetro │──────▶ │ Guardia Falsa Alarma  │
  sensores           │ (ignora el movimiento│        │ Mascota (se autoco-  │
                    │  en modo Parcial)     │        │  rrige si el modo    │
                    └──────────┬───────────┘        │  es erróneo + la     │
                               │                     │  mascota activa la   │
                               ▼                     │  sirena)             │
                    ┌─────────────────────┐        └───────────────────────┘
                    │ Rearme (limpieza auto)│
                    └─────────────────────┘
```

## Blueprints en este repositorio

| Blueprint | Dominio | Propósito |
|---|---|---|
| `arm.yaml` | script | Arma el sistema; elige automáticamente Completo/Parcial según los sensores de mascota, salvo que se indique lo contrario |
| `disarm.yaml` | script | Desarma; deshace los efectos secundarios de la sirena si realmente estaba sonando |
| `person_trigger.yaml` | automatización | Una instancia por persona; activa/desactiva el armado desde un sensor de ID numérico |
| `perimeter_trigger.yaml` | automatización | Hace sonar la sirena ante una brecha; consciente del modo |
| `rearm.yaml` | automatización | Limpia la sirena una vez que los sensores vuelven al estado de reposo |
| `auto_arm_away.yaml` | automatización | Arma automáticamente cuando los rastreadores de presencia muestran que todos están fuera |
| `pet_false_alarm_guard.yaml` | automatización | Se autocorrige ante una falsa alarma causada por la mascota, y siempre te notifica |

Consulta [`docs/hardware.md`](../hardware.es.md) para notas sobre qué hardware importa de verdad (cámaras/GPU, dispositivo de autenticación, sirena, satélites de voz, VPN para presencia fiable fuera de casa).

## Requisitos previos

- Una forma de autenticar personas con un ID numérico por persona (un lector de huellas Zigbee/Tuya que expone un `sensor.*` con el ID coincidente es la implementación de referencia con la que se construyó esto; un teclado con códigos mapeados a números, o un lector NFC, funcionarían igual).
- Una entidad de sirena `switch.*`.
- Un `input_boolean` para el estado "armado".
- Una segunda entidad de tipo booleano para el indicador de "modo Parcial". Idealmente un segundo `input_boolean` — **pero** si tu instancia no te permite crear nuevas entidades helper desde las herramientas de automatización (nos pasó a nosotros: la API REST de configuración de Home Assistant permite crear/editar automatizaciones y scripts, pero no helpers `input_boolean`/`input_select`), puedes reutilizar cualquier entidad `automation` de repuesto solo por su estado on/off. Es un truco, pero funciona bien y es invisible para el usuario final — mira `blueprints/script/ha-smart-perimeter/arm.yaml` para ver cómo se gestiona el on/off de forma genérica mediante `homeassistant.turn_on`/`turn_off`, de modo que funciona con ambos tipos de entidad.
- (Opcional, para el modo consciente de mascotas) Una canalización de IA en cámara que te dé un sensor binario por cámara de "mascota detectada". Con Frigate, esto significa añadir `dog` (o la especie que tengas) a `objects.track` en la configuración de tu cámara — no hace falta entrenar ningún modelo personalizado, `dog` ya es una de las 80 clases estándar de COCO que soportan la mayoría de los modelos detectores de Frigate.
- (Opcional, para el auto-armado al salir) Presencia fiable mediante `person.*`/`device_tracker.*` — mira la nota en `auto_arm_away.yaml` sobre por qué una VPN/acceso remoto es un requisito para que esto funcione de verdad cuando sales de tu red Wi-Fi.

## Instalación

1. En Home Assistant: Ajustes → Automatizaciones y escenas → Blueprints → Importar Blueprint, y pega la URL raw de GitHub de cada archivo `.yaml` de este repositorio (o simplemente copia los archivos en tu estructura de carpetas `config/blueprints/...` y recarga).
2. Crea el `input_boolean` para el estado armado (e, idealmente, un segundo para el modo parcial) en Ajustes → Dispositivos y servicios → Helpers.
3. Crea un script a partir de `arm.yaml` y otro de `disarm.yaml`, rellenando tus entidades.
4. Crea una automatización `person_trigger.yaml` **por cada persona** que deba poder armar/desarmar.
5. Crea automatizaciones a partir de `perimeter_trigger.yaml`, `rearm.yaml` y `pet_false_alarm_guard.yaml`. **Déjalas desactivadas al principio** — el script Armar las activa/desactiva por ti; no deberían estar habilitadas de forma independiente fuera de una sesión armada.
6. Opcionalmente, `auto_arm_away.yaml` para un armado automático sin intervención cuando todos salen.

## Lecciones aprendidas (leer antes de confiar en esto para seguridad real)

- **Las comprobaciones de presencia instantáneas son frágiles.** Nuestra primera versión comprobaba "¿la mascota es visible ahora mismo?" en el instante exacto del armado, y solía equivocarse — la mascota simplemente estaba fuera de cuadro durante un minuto. El script `arm.yaml` en cambio comprueba una **ventana de tiempo** (mascota vista en los últimos N minutos, por defecto 15), mucho más tolerante. La automatización `pet_false_alarm_guard.yaml` hace deliberadamente lo contrario (comprobación instantánea) porque ahí la pregunta es distinta: "¿está la mascota causando este disparo específico ahora mismo?".
- **"Hay alguien en casa" basado en cámaras no es lo mismo que basarse en el teléfono.** Las cámaras solo cubren las habitaciones hacia las que apuntan. Al principio usábamos "ninguna persona visible en ninguna cámara" como disparador para el auto-armado, y armó la casa mientras un familiar seguía dentro, simplemente en una habitación sin cobertura — una falsa alarma real y en vivo. `auto_arm_away.yaml` está construido específicamente en torno a la presencia `person`/`device_tracker` para evitar esto; no lo sustituyas por sensores de ocupación de cámara.
- **Dos dispositivos de armado/desarmado independientes necesitan una única fuente de verdad.** Si tienes más de una forma de armar (p. ej. un lector de huellas *y* un mando a distancia tipo llavero), haz que ambos pasen por los mismos scripts Armar/Desarmar en lugar de escribir lógica separada para cada dispositivo — de lo contrario, el helper "armado" y el estado real del sistema pueden desincronizarse silenciosamente.
- **Una comprobación periódica en segundo plano de "¿la mascota realmente se fue?" es una decisión de compromiso real, no una mejora gratuita.** Consideramos añadir una re-comprobación periódica que pasara automáticamente de Parcial a Completo una vez que la mascota no se hubiera visto en un rato, y deliberadamente no la incluimos aquí como comportamiento por defecto: usa exactamente la misma señal frágil de "la cámara la ve" que causó el error original, solo que en la dirección contraria. Si la añades tú mismo, usa una ventana de "definitivamente se fue" notablemente más amplia (sugerimos 30-40 minutos, no 15) que la comprobación en el momento del armado, ya que un paso erróneo de Parcial a Completo reintroduce falsas alarmas, mientras que quedarse en Parcial algo más de tiempo solo cuesta una cobertura de movimiento reducida.

## Licencia

MIT — ver `LICENSE`.
