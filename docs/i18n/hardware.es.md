[English](../hardware.md) | [Italiano](hardware.it.md) | [Español](hardware.es.md) | [日本語](hardware.ja.md)

# Notas sobre el hardware

Este proyecto es independiente del hardware — los blueprints solo dependen de los tipos de entidad, no de la marca. Esta página recoge lo que realmente importó al construir y usar la instalación de referencia, para que puedas comprar con criterio en lugar de adivinar.

## Cámaras / detección con IA

- Sirve cualquier cámara compatible con Frigate (o una canalización de detección de objetos equivalente). Lo que importa para la función "pet-aware" es la **decodificación de vídeo por hardware (NVDEC o equivalente)**, no la potencia de cálculo bruta de la GPU — incluso una GPU antigua y barata puede decodificar varias transmisiones en paralelo y ejecutar el detector con suficiente rapidez.
- Si además transcodificas una transmisión para verla desde el navegador o de forma remota (por ejemplo con go2rtc, algo habitual en cámaras que no hablan de forma nativa un códec compatible con el navegador), ese es un coste **aparte** de la detección, y por software/CPU se vuelve caro enseguida. Una GPU con **codificación por hardware (NVENC o equivalente)** quita esa carga a la CPU. Las GPU baratas suelen tener decodificación pero no codificación: antes de comprar una para esto, comprueba ambas especificaciones, no solo si "tiene GPU".
- Detectar una mascota no requiere entrenamiento ni modelos personalizados: especies como "perro" y "gato" ya forman parte del conjunto estándar de 80 clases COCO que incluyen la mayoría de los modelos de detección de Frigate — basta con añadir la especie a la lista de objetos rastreados de la cámara.

## Dispositivo de autenticación (huella / teclado / NFC)

- Sirve cualquier dispositivo que termine exponiendo en Home Assistant un `sensor.*` con "qué persona coincidió" como ID numérico o de texto: funciona como disparador de armado/desarmado. La implementación de referencia usa un lector de huellas Zigbee/Tuya.
- Si el dispositivo tiene un LED de estado direccionable, conectarlo a una entidad `light.*` de HA es un detalle útil para la instalación o el feedback de errores, pero ningún blueprint de aquí lo requiere.

## Sirena

- Basta un simple relé o un enchufe inteligente que accione una sirena estándar de 12V — no hace falta un producto de "sirena de alarma" específico. Los blueprints solo llaman a `switch.turn_on` / `switch.turn_off`.

## Avisos por voz

- Sirve cualquier entidad `assist_satellite.*` — satélites de voz basados en ESPHome, el propio hardware de voz de Home Assistant, etc. Los blueprints avisan a una **lista** de device IDs, así que puedes apuntar a varios satélites a la vez, y si uno está desconectado el sistema sigue funcionando con normalidad.

## Presencia fiable fuera de casa

- La presencia basada en el teléfono (`person.*` / `device_tracker.*`) solo se mantiene precisa fuera del Wi-Fi de casa si el teléfono conserva una conexión activa con tu instancia de Home Assistant. Sin eso, la presencia se queda congelada en el "último estado conocido" en el instante en que el teléfono sale del Wi-Fi, lo que rompe silenciosamente `auto_arm_away.yaml`.
- Una VPN siempre activa soluciona esto. No hace falta nada complicado: en el lado de red basta un router doméstico con **cliente y servidor WireGuard integrados** (sin necesidad de un add-on aparte). En el teléfono, activa la opción del sistema "VPN siempre activa" en lugar de confiar en una app de automatización para encender/apagar una VPN de terceros — la mayoría de las apps de automatización solo pueden *abrir* la app de VPN, no iniciar o detener su túnel de forma fiable.

## Hardware mínimo necesario

- Una GPU con decodificación de vídeo por hardware, solo si quieres la función "pet-aware" — si no, no hace falta en absoluto.
- Un sensor de autenticación por cada punto de acceso (huella / teclado / NFC).
- Los sensores de puerta/ventana/movimiento que ya tengas — no se necesita nada específico de alarmas, las entidades `binary_sensor` estándar de HA funcionan tal cual.
- Una sirena accionada por relé.
- Todo lo demás (avisos por voz, armado automático al salir) es opcional y adicional.
