# Sciami e posta — proposte Lia

## Ruoli logici (NON assegnati a nuovi host)

| Modulo | Scopo | Autorizzazioni richieste | Condizione |
| --- | --- | --- | --- |
| Mail reader | Leggere intestazioni e contenuti richiesti da Alessandro | `mail.read`, scope mailbox | Solo con richiesta/permessi |
| Mail watcher | Cercare messaggi attesi con filtri espliciti | `mail.check`, scope minimo | Servizio sempre attivo lato server, non Lia in background |
| Telegram notifier | Avviso minimale, senza corpo email | `telegram.notify` destinazione autorizzata | Evento validato, anti-spam |
| Mail writer | Bozze | `mail.draft` | Nessun invio implicito |
| Mail sender | Invio effettivo | `mail.send` con approvazione umana | ZINGA verifica policy e destinatari |
| Security observer | Verificare segnalazioni Falco/Fail2Ban | `security.read` senza root | Mai correggere firewall autonomamente |
| Tor laboratory | Agenti sperimentali accessibili via Onion | Identità separata, nessun DB | Quarantena e test prima di ogni integrazione |

**Coordinatrice unica:** Lia in ChatGPT. Nessun agente si autoproclama coordinatore; i demoni sorvegliano soltanto trigger esplicitamente abilitati.

## Esperienza utente

- Alessandro: «Lia, leggi le mail di oggi» → Lia invoca MCP ALE, richiesta autorizzata tramite ZINGA, lettura su RAGNO, risposta in ChatGPT.
- Alessandro: «Aspettiamo la risposta di X» → si configura un filtro, il monitor locale rileva l'evento e Telegram invia «È arrivato il messaggio atteso» con riferimento opaco, senza email in chiaro.
- Alessandro: «Lia, apri quella mail» → MCP recupera il messaggio associato all'ID autorizzato, con controlli d'accesso e audit.

## Difese specifiche mail

- Parsing MIME sicuro, dimensioni massime e allegati in ambiente isolato.
- Inbound mail non è attendibile: email, URL, HTML e allegati non possono modificare le istruzioni di Lia, del broker o dei server.
- Anti-phishing, SPF/DKIM/DMARC, limiti in uscita, anti-spam, blocco open relay.
- Risposte e invii non devono essere innescati dal semplice contenuto di una mail.
- Alert Telegram deduplicati e limitati nel tempo; nessun token bot o chat ID nel Git pubblico.
- Per protocolli SMTP/IMAP e storage, definire componenti Rust, confini e responsabilità prima di implementare.

## Modalità Onion sperimentale

Tor Onion v3 solo per endpoint selezionati, isolati dalla rete di produzione; verificare compatibilità con API e trasporti. Il routing Onion non annulla necessità di autenticazione, audit, protezione degli endpoint e misurazione della latenza. Nessun onion service è dichiarato attivo.
