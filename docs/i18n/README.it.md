[English](../../README.md) | [Italiano](README.it.md) | [Español](README.es.md) | [日本語](README.ja.md)

# Smart Perimeter

Un sistema di allarme/perimetro per Home Assistant costruito con dei blueprint, con due caratteristiche che la maggior parte degli allarmi fai-da-te non ha:

1. **Armo/disarmo multi-persona** da qualsiasi sensore "chi si è autenticato" con ID numerico (lettore di impronte digitali, tastierino, lettore NFC...), con un'unica fonte di verità per lo stato armato — quindi non importa quale dispositivo o quale persona lo abbia attivato, lo stato non si disallinea mai.
2. **Modalità parziale consapevole degli animali domestici.** Se hai un animale che resta a casa da solo, i sensori di movimento di un allarme normale scatteranno in continuazione per colpa sua. Questo sistema usa un rilevatore AI di oggetti su telecamera (testato con il tracciamento dell'oggetto "dog" di [Frigate](https://frigate.video/), ma funziona con qualsiasi integrazione che dia un sensore binario "animale nell'inquadratura" per telecamera) per armarsi automaticamente in modalità **Completa** (tutti i sensori) quando non c'è nessuno — né persone né animali — oppure in modalità **Parziale** (solo porte/finestre, niente movimento interno) quando l'animale è a casa da solo.

Tutto qui è impacchettato come [Blueprint di Home Assistant](https://www.home-assistant.io/docs/automation/using_blueprints/): li importi, compili le tue entità, fatto. Nessuna modifica manuale allo YAML necessaria per un setup di base.

## Perché non usare semplicemente un'integrazione di allarme normale?

Puoi farlo, e per le basi (porta/sirena) probabilmente dovresti riusare quello che già ti offre l'integrazione del tuo pannello d'allarme. Questo progetto esiste per colmare il vuoto specifico che ha la maggior parte degli allarmi: **non sanno distinguere il tuo animale domestico da un intruso**, quindi o disattivi permanentemente i sensori di movimento interni (sicurezza più debole) o convivi con falsi allarmi ogni volta che l'animale si muove.

## Architettura

```
                    ┌─────────────────────┐
  impronta/    ───▶│ Trigger Persona       │
  tastierino/NFC    │ (uno per persona)     │
                    └──────────┬───────────┘
                               │ chiama
                               ▼
                    ┌─────────────────────┐        ┌───────────────────────┐
  presenza   ──────▶│   Script Arma        │───────▶│ Script Disarma        │
  (auto-arm)        │ (decide Completa/     │◀───────│                       │
                    │  Parziale dai sensori │        └───────────────────────┘
                    │  telecamera animale)  │
                    └──────────┬───────────┘
                               │ abilita
                               ▼
                    ┌─────────────────────┐        ┌───────────────────────┐
  movimento/porta ─▶│ Trigger Perimetro    │──────▶ │ Guardia Falso Allarme │
  sensori           │ (ignora il movimento │        │ Animale (si autocor-  │
                    │  in modalità Parziale)│        │  regge se modalità    │
                    └──────────┬───────────┘        │  sbagliata + animale  │
                               │                     │  attiva la sirena)    │
                               ▼                     └───────────────────────┘
                    ┌─────────────────────┐
                    │ Riarmo (pulizia auto) │
                    └─────────────────────┘
```

## Blueprint in questo repository

| Blueprint | Dominio | Scopo |
|---|---|---|
| `arm.yaml` | script | Arma il sistema; sceglie automaticamente Completa/Parziale dai sensori animale, a meno che non venga specificato diversamente |
| `disarm.yaml` | script | Disarma; annulla gli effetti collaterali della sirena se stava effettivamente suonando |
| `person_trigger.yaml` | automazione | Un'istanza per persona; attiva/disattiva l'armo da un sensore con ID numerico |
| `perimeter_trigger.yaml` | automazione | Fa suonare la sirena in caso di violazione; consapevole della modalità |
| `rearm.yaml` | automazione | Ripulisce la sirena quando i sensori tornano allo stato di riposo |
| `auto_arm_away.yaml` | automazione | Arma automaticamente quando i tracker di presenza mostrano tutti fuori casa |
| `pet_false_alarm_guard.yaml` | automazione | Si autocorregge in caso di falso allarme causato dall'animale, e ti avvisa sempre |

Vedi [`docs/hardware.md`](hardware.it.md) per note su quale hardware conta davvero (telecamere/GPU, dispositivo di autenticazione, sirena, satelliti vocali, VPN per una presenza affidabile fuori casa).

## Prerequisiti

- Un modo per autenticare le persone con un ID numerico per persona (un lettore di impronte digitali Zigbee/Tuya che espone un `sensor.*` con l'ID corrispondente è l'implementazione di riferimento usata per costruire questo progetto; un tastierino con codici mappati a numeri, o un lettore NFC, funzionerebbero allo stesso modo).
- Un'entità sirena `switch.*`.
- Un `input_boolean` per lo stato "armato".
- Una seconda entità booleana per il flag "modalità Parziale". Idealmente un secondo `input_boolean` — **ma** se la tua istanza non ti permette di creare nuove entità helper dagli strumenti di automazione (è successo a noi: l'API REST di configurazione di Home Assistant supporta la creazione/modifica di automazioni e script, ma non degli helper `input_boolean`/`input_select`), puoi riusare qualsiasi entità `automation` di scorta solo per il suo stato on/off. È un espediente, ma funziona bene ed è invisibile all'utente finale — vedi `blueprints/script/ha-smart-perimeter/arm.yaml` per come l'on/off viene gestito in modo generico tramite `homeassistant.turn_on`/`turn_off`, così funziona con entrambi i tipi di entità.
- (Opzionale, per la modalità consapevole degli animali) Una pipeline AI su telecamera che dia un sensore binario per telecamera per "animale rilevato". Con Frigate, significa aggiungere `dog` (o qualsiasi specie tu abbia) a `objects.track` nella configurazione della telecamera — nessun modello personalizzato/addestramento necessario, `dog` è già una delle 80 classi COCO standard supportate dalla maggior parte dei modelli detector di Frigate.
- (Opzionale, per l'armo automatico quando si esce) Presenza affidabile via `person.*`/`device_tracker.*` — vedi la nota in `auto_arm_away.yaml` sul fatto che una VPN/accesso remoto sia un prerequisito perché questo funzioni davvero quando esci dalla tua rete Wi-Fi.

## Installazione

1. In Home Assistant: Impostazioni → Automazioni e scene → Blueprint → Importa Blueprint, e incolla l'URL raw di GitHub di ciascun file `.yaml` di questo repository (oppure copia semplicemente i file nella struttura di cartelle `config/blueprints/...` e ricarica).
2. Crea l'`input_boolean` per lo stato armato (e, idealmente, un secondo per la modalità parziale) sotto Impostazioni → Dispositivi e servizi → Helper.
3. Crea uno script da `arm.yaml` e uno da `disarm.yaml`, compilando le tue entità.
4. Crea un'automazione `person_trigger.yaml` **per ogni persona** che deve poter armare/disarmare.
5. Crea automazioni da `perimeter_trigger.yaml`, `rearm.yaml` e `pet_false_alarm_guard.yaml`. **Lasciale disattivate all'inizio** — lo script Arma le accende/spegne per te; non dovrebbero essere abilitate in modo indipendente al di fuori di una sessione armata.
6. Opzionalmente, `auto_arm_away.yaml` per l'armo automatico senza intervento quando tutti escono.

## Lezioni imparate (leggi prima di affidarti a questo per la sicurezza reale)

- **I controlli di presenza istantanei sono fragili.** La nostra prima versione controllava "l'animale è visibile proprio ora" nell'esatto momento dell'armo, e spesso sbagliava — l'animale si trovava semplicemente fuori dall'inquadratura per un minuto. Lo script `arm.yaml` controlla invece una **finestra temporale** (animale visto negli ultimi N minuti, default 15), molto più tollerante. L'automazione `pet_false_alarm_guard.yaml` fa deliberatamente il contrario (controllo istantaneo) perché lì la domanda è diversa: "l'animale sta causando proprio ora questo specifico allarme".
- **"C'è qualcuno in casa" basato sulle telecamere non è lo stesso di basarsi sul telefono.** Le telecamere coprono solo le stanze verso cui sono puntate. Inizialmente usavamo "nessuna persona visibile da nessuna telecamera" come trigger per l'armo automatico, e ha armato la casa mentre un familiare era ancora dentro, semplicemente in una stanza scoperta — un vero falso allarme reale. `auto_arm_away.yaml` è costruito specificamente attorno alla presenza `person`/`device_tracker` per evitare questo; non sostituirlo con sensori di occupazione da telecamera.
- **Due dispositivi di armo/disarmo indipendenti hanno bisogno di un'unica fonte di verità.** Se hai più di un modo per armare (es. un lettore di impronte digitali *e* un telecomando a portachiavi), fai passare entrambi attraverso gli stessi script Arma/Disarma invece di scrivere logica separata per ogni dispositivo — altrimenti l'helper "armato" e lo stato reale del sistema possono disallinearsi silenziosamente.
- **Un controllo periodico in background per "l'animale è davvero uscito" è un compromesso reale, non un aggiornamento gratuito.** Abbiamo considerato di aggiungere un ricontrollo periodico che passerebbe automaticamente da Parziale a Completa una volta che l'animale non viene visto da un po', e deliberatamente non l'abbiamo aggiunto qui come comportamento predefinito: usa esattamente lo stesso segnale fragile "la telecamera lo vede" che ha causato il bug originale, solo nella direzione opposta. Se lo aggiungi tu stesso, usa una finestra "sicuramente andato via" nettamente più larga (suggeriamo 30-40 minuti, non 15) rispetto al controllo al momento dell'armo, dato che un passaggio errato da Parziale a Completa reintroduce falsi allarmi, mentre restare in Parziale un po' troppo a lungo costa solo una copertura di movimento ridotta.

## Licenza

MIT — vedi `LICENSE`.
