[English](../hardware.md) | [Italiano](hardware.it.md) | [Español](hardware.es.md) | [日本語](hardware.ja.md)

# Note sull'hardware

Questo progetto è indipendente dall'hardware — i blueprint si basano solo sui tipi di entità, non sulle marche. Questa pagina raccoglie ciò che davvero ha fatto la differenza costruendo e usando l'impianto di riferimento, così puoi scegliere con criterio invece che a caso.

## Telecamere / rilevamento AI

- Va bene qualsiasi telecamera supportata da Frigate (o una pipeline di object detection equivalente). Per la funzione "pet-aware" conta soprattutto la **decodifica video hardware (NVDEC o equivalente)**, non la potenza di calcolo grezza della GPU — anche una GPU vecchia ed economica riesce a decodificare più flussi in parallelo e a far girare il rilevatore abbastanza velocemente.
- Se transcodifichi anche un flusso per la visione da browser/remoto (es. con go2rtc, spesso necessario per telecamere che non parlano nativamente un codec compatibile col browser), quello è un costo **separato** dal rilevamento, e in software/CPU diventa costoso in fretta. Una GPU con **encoding hardware (NVENC o equivalente)** toglie quel carico dalla CPU. Le GPU economiche spesso hanno la decodifica ma non l'encoding: prima di comprarne una per questo scopo, controlla entrambe le specifiche, non solo "ha una GPU".
- Rilevare un animale domestico non richiede training o modelli personalizzati: specie come "cane" e "gatto" fanno già parte del set standard di 80 classi COCO che la maggior parte dei modelli di rilevamento di Frigate include di serie — basta aggiungere la specie alla lista degli oggetti tracciati della telecamera.

## Dispositivo di autenticazione (impronta / tastierino / NFC)

- Va bene qualsiasi dispositivo che finisca per esporre in Home Assistant un `sensor.*` con "quale persona ha corrisposto" come ID numerico/testuale: funziona come trigger di armo/disarmo. L'implementazione di riferimento usa un lettore di impronte Zigbee/Tuya.
- Se il dispositivo ha un LED di stato indirizzabile, collegarlo a un'entità `light.*` di HA è un tocco in più utile per il feedback durante l'installazione o in caso di errore, ma nessun blueprint qui lo richiede.

## Sirena

- Basta un semplice relè o uno smart switch che pilota una normale sirena a 12V — non serve un prodotto "sirena da antifurto" dedicato. I blueprint chiamano solo `switch.turn_on` / `switch.turn_off`.

## Annunci vocali

- Va bene qualsiasi entità `assist_satellite.*` — satelliti vocali basati su ESPHome, l'hardware vocale nativo di Home Assistant, ecc. I blueprint annunciano su una **lista** di device ID, quindi puoi puntare a più satelliti insieme, e se uno di loro è offline il sistema si comporta comunque correttamente.

## Presenza affidabile fuori casa

- La presenza basata sul telefono (`person.*` / `device_tracker.*`) resta accurata anche fuori dal Wi-Fi di casa solo se il telefono mantiene una connessione attiva verso la tua istanza Home Assistant. Senza questo, la presenza si blocca sull'"ultimo stato noto" nell'istante in cui il telefono esce dal Wi-Fi — cosa che rompe silenziosamente `auto_arm_away.yaml`.
- Una VPN sempre attiva risolve il problema. Non serve niente di complicato: sul lato rete basta un router consumer con **client+server WireGuard integrati** (senza bisogno di un add-on separato). Sul telefono, attiva l'impostazione di sistema "VPN sempre attiva" invece di affidarti a un'app di automazione per accendere/spegnere una VPN di terze parti — la maggior parte delle app di automazione riesce solo ad *aprire* l'app VPN, non ad avviarne/fermarne davvero il tunnel in modo affidabile.

## Hardware minimo indispensabile

- Una GPU con decodifica video hardware, solo se vuoi la funzione "pet-aware" — altrimenti non serve affatto.
- Un sensore di autenticazione per ogni punto di accesso (impronta / tastierino / NFC).
- I sensori porta/finestra/movimento che già hai — non serve nulla di specifico per antifurti, le normali entità `binary_sensor` di HA vanno bene così come sono.
- Una sirena pilotata da relè.
- Tutto il resto (annunci vocali, armo automatico quando esci) è opzionale e aggiuntivo.
