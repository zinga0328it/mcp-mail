# Resilienza e ripristino — proposta Lia

**Progettazione soltanto.** Definire obiettivi RPO/RTO con Alessandro prima della produzione.

## Isolamento dei guasti

| Guasto | Comportamento sicuro | Ripristino |
| --- | --- | --- |
| RAGNO spento | Il gateway segnala mail non disponibile; nessun bypass delle autorizzazioni | Host sostitutivo con build Rust firmata e config versionata; restore mail queue |
| ZINGA irraggiungibile | Nuove azioni privilegiate e accessi DB **negati**, anche se ALE è vivo | Recupero con credenziali break-glass fuori banda e rotazione; controllare audit |
| ALE MCP fermo | ChatGPT non interroga gli agenti; il monitor mail locale può notificare da solo se separato | Ripristinare unità MCP, DNS/TLS, connettore, test su staging |
| ALE PostgreSQL guasto | Persistenza sospesa, eventuale spool locale cifrato e limitato; niente accesso DB alternativo | Backup completo + WAL/PITR, verifica consistenza, ripartenza controllata |
| Yggdrasil interrotto | Nessuna apertura automatica su Internet; agenti in attesa o degradati | Verificare peer/routing, test del percorso secondario preventivamente approvato |
| Cloudflare Tunnel fermo | Endpoint MCP esterno non disponibile, non si cambia policy | Riparare tunnel/identità; maintenance per altra via autorizzata |
| Telegram non funzionante | Evento trattenuto in coda con scadenza e deduplicazione; evitare dati riservati | Retry con backoff, verifica webhook/bot e alert alternativo |
| Falco/Fail2Ban difettoso | Visibilità ridotta, allerta manutenzione; non spegnere firewall come soluzione | Ripristinare ultima configurazione verificata |
| Tor/Onion indisponibile | Nessun impatto su percorso primario | Disattivare modulo sperimentale e fare diagnosi isolata |

## Backup e recupero dei dati

- PostgreSQL su ALE: backup cifrati consistenti, WAL archiviati se PITR richiesto; copia indipendente fuori da ALE, altrimenti la rottura fisica elimina sia DB sia backup.
- Backup configurazione per ogni host, senza segreti nel repository Git; credenziali e chiavi in vault/offline con procedura separata.
- Rotazione dei supporti, verifica hash, test automatizzati del restore su ambiente isolato; nessuna promessa di ripristino senza test reali.
- Mail: strategia di snapshot di queue/spool e criteri di conservazione/retention, senza cancellare un messaggio confermato prima del commit durevole.
- Aggiornamenti come rilascio atomico, con versioni note-buone e rollback.

## Contratti dei moduli

Ogni agente/modulo dichiara:
- Versione, maintainer, host e account dedicato.
- Input/output documentati, deadline e timeout.
- Permessi del broker, identità mTLS, scope e TTL.
- Endpoint di health, stato ready, metriche e errori tipizzati.
- Idempotency key per azioni ripetibili; retry con jitter e backoff limitati, circuit breaker.
- Audit, diagnostica senza segreti, e test di compatibilità.
- Come disabilitarlo e tornare alla versione precedente senza modificare altri moduli.

## Primo esperimento di continuità

1. In staging: simula fermo di RAGNO mentre un'email è attesa.
2. Verifica che Lia riceva errore chiaro e che nessun agente usi scorciatoie.
3. Avvia una versione sostitutiva del modulo; nessun accesso nuovo al DB.
4. Ripristina queue/spool e verifica nessuna duplicazione grazie agli ID.
5. Simula ZINGA fuori servizio: verificare deny e log.
6. Ripristina ZINGA via accesso autorizzato fuori banda; ruota eventuali credenziali esposte.
7. Registra tempo misurato, perdita dati e azioni manuali realmente necessarie.

**Regola generale:** un recupero non deve diventare un bypass permanente del broker.
