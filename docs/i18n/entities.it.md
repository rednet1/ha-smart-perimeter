[English](../entities.md) | [Italiano](entities.it.md) | [Español](entities.es.md) | [日本語](entities.ja.md)

# Scheda di configurazione

Compila questa scheda con le tue entità prima di importare i blueprint — è più comodo averle tutte in un posto solo piuttosto che cercarle una alla volta mentre clicchi nel modulo di input di ogni blueprint.

| Ruolo | Dominio d'esempio | La tua entità |
|---|---|---|
| Helper stato armato | `input_boolean.*` | |
| Flag modalità parziale | `input_boolean.*` o `automation.*` di scorta | |
| Sirena | `switch.*` | |
| LED / indicatore visivo (opzionale) | `light.*` | |
| Sensore ID di autenticazione (impronta/tastierino/NFC) | `sensor.*` | |
| Sensori di movimento | `binary_sensor.*` (device_class: motion) | |
| Sensori porta/finestra | `binary_sensor.*` (device_class: door/window/opening) | |
| Sensori occupazione animale (per telecamera) | `binary_sensor.*` | |
| Tracker di presenza (per l'armo automatico) | `person.*` / `device_tracker.*` | |
| Destinatari annunci vocali (opzionale) | `assist_satellite.*` | |

Tabella per persona (una riga → un'istanza dell'automazione `person_trigger.yaml`):

| Persona | ID sul sensore | Note |
|---|---|---|
| | | |
| | | |
