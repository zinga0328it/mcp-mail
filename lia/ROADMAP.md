# Roadmap sperimentale di Lia

Questa roadmap NON concede permessi di installazione o modifica in produzione.

| Fase | Progetto | Esito verificabile |
| --- | --- | --- |
| 0 | Inventario ALE/ZINGA/RAGNO/ALEX/AAA, Linux, storage e rete | Tabella verificata con Alessandro |
| 1 | Policy broker ZINGA, ruoli, chiavi, log e revoca | Azione senza permesso negata; audit senza segreti |
| 2 | Zone di rete Yggdrasil, nftables v4/v6, accesso SSH | Test ACL e rollback automatico validati |
| 3 | Endpoint MCP ALE cifrato e autenticato | Lia legge solo gli strumenti esplicitamente concessi |
| 4 | RAGNO Rust mail + DB tramite broker | Nessun accesso PostgreSQL diretto; test lettura autorizzata |
| 5 | Monitor mail locale + Telegram | Evento filtrato, notifica deduplicata, privacy verificata |
| 6 | Falco/Fail2Ban con suite di test, backup e restore | Alert misurati e restore isolato completato |
| 7 | Canali opzionali ZeroTier e Tor Onion | Prove di isolamento senza modificare percorso primario |
| 8 | Valutazione OpenStack se hardware adeguato | Stima risorse e costi, laboratorio ricostruibile |

## Gate di rilascio

- Approvazione esplicita Alessandro.
- Threat model, access matrix e test di regressione aggiornati.
- Backup ripristinabile verificato, rollback pronto.
- Nessuna credenziale in Git, no listener nuovo non autorizzato.
- Verifica pacchetti e dipendenze, health check e allarmi.
- Funzionamento sicuro quando broker e overlay sono offline.

## Cose da non fare

- Non trasferire automaticamente le proposte del ramo Lia verso `Alessandro` o `main`.
- Non eseguire cambi di `nftables`/SSH in remoto senza finestra e accesso di recupero.
- Non presumere che una SIM sia anonima o garantisca anonimato.
- Non creare un cluster OpenStack sulle macchine finché non si confermano CPU, RAM, dischi e utilità reale.
- Non affidare al software LLM chiavi root o shell arbitraria.
