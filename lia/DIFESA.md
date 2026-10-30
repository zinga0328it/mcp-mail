# Difesa profonda — proposta Lia

**Solo progettazione; nessun comando eseguito sui server.**

## Baseline Linux: configurazione dichiarativa e ripetibile

- Servizi separati per utente Linux, namespace/VM ove appropriato, filesystem di sola lettura e capability minime; unit systemd con sandboxing, riavvio e limiti risorse.
- Aggiornamenti di sicurezza pianificati, inventario pacchetti e SBOM, build Rust riproducibili quando pratico; pin delle dipendenze e firma degli artefatti.
- SSH con chiavi hardware o chiavi moderne, no password, no root diretto, accesso da indirizzi di gestione, MFA nel sistema di ingresso e `sudo` tracciato.
- Credenziali consegnate da ZINGA con TTL minimo; rotazione, revoca e identita per ogni agente. Nessun segreto in ambiente di test, repository, log o immagini.
- TLS anche sopra Yggdrasil per le API sensibili; pin dell'identità del servizio o mTLS dove sostenibile.
- Isolare posta in ingresso come **input non fidato**; HTML sanitizzato, allegati trattati in sandbox, nessun comando eseguito a partire da contenuti ricevuti.

## Controlli che conosciamo

| Strumento | Compito specifico | Controllo |
| --- | --- | --- |
| nftables | Default deny, egress allowlist, ACL per interfacce IPv4/IPv6, rate limit | Validare `nft -c -f` e rollback temporizzato; non tagliare SSH |
| Fail2Ban | Bloccare tentativi abusivi identificati dai log | Jail e ban backend nftables; verificare log parser e sblocco |
| Falco | Allarme comportamenti di runtime, processi e syscall sospette | Regole mirate, riduzione falsi positivi, notifiche senza segreti |
| Apache2 | Reverse proxy web di ALEX | TLS, header, path allowlist; amministrazione non esposta |
| SSH | Manutenzione | Accesso a chiave, controlli di origine, MFA dove possibile |
| Cloudflare Tunnel | Ingresso HTTPS controllato verso MCP | Autenticazione applicativa distinta; evitare API di admin pubbliche |
| Tor Onion | Endpoint sperimentali v3 isolati | Client authorization ove applicabile, autentica applicativa, niente DB diretto |
| Yggdrasil | Overlay principale | Non è una ACL: nftables su ygg0, identità dei peer applicativi |
| ZeroTier | Canale opzionale di break-glass | Reti e identità strettamente autorizzate, non sempre attivo |
| OpenStack | Eventuale laboratorio isolato/VM | Prima verificare RAM, storage e requisiti operativi |
| SIM dedicate | Ridondanza di connessione lecita | Nessuna SIM equivale a un fattore di autenticazione |

## Matrice di accesso: bozza da validare

- ChatGPT/Lia → ingresso MCP ALE: solo strumenti autenticati, no shell generica.
- MCP ALE → ZINGA: richieste con identità, contesto, azione, motivazione, scadenza; niente credenziali database.
- ZINGA → PostgreSQL ALE: solo ruolo DB del broker, su Yggdrasil, connessione autenticata; traffico DB negato dagli altri host.
- RAGNO → ZINGA: API specifiche per lettura/scrittura poste; niente PostgreSQL diretto.
- RAGNO ↔ Internet: solo porte necessarie a SMTP e client mail, o relay; egress con controllo.
- ALEX → altri nodi: solo destinazioni esplicitamente autorizzate, mai transito universale.
- AAA: nessuna autorizzazione di nuovo servizio finché Alessandro non assegna il ruolo.
- Servizi sperimentali Onion → infrastruttura: nessuna accesso a ZINGA/DB salvo autorizzazioni circoscritte.

## Tre difese contro compromissione agente

1. **Capability ristrette**: un token `mail.read:folder/inbox` non permette `mail.send` né amministrazione.
2. **Conferma umana**: invii, eliminazioni, accessi anomali o configurazioni in produzione richiedono l'approvazione prevista.
3. **Registrazione controllata**: tracce di decisione del broker con request ID, azione, policy, esito e timestamp, senza contenuti privati.

## Test periodici previsti

- `nft` policy fail-closed e rollback d'emergenza.
- Tentativo di connessione DB da RAGNO/ALEX/AAA deve essere negato.
- Revoca token e accesso negato a operatore non autorizzato.
- Simulazione di allegato malevolo e prompt injection mail; nessuna esecuzione.
- Allarme Falco e sblocco Fail2Ban testato senza interrompere il servizio.
- Rotazione chiavi e ripristino con doppia verifica.

**Principio:** la sicurezza non aumenta automaticamente accumulando tunnel. Ogni hop e ogni controllo deve avere chi lo gestisce, chi lo verifica e come ripristinarlo.
