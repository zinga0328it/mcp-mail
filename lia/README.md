# LIA — laboratorio di architettura MCP SWARM 🐝

**Ramo:** `Lia` · **Autore proposte:** Lia · **Architetto/approvazione:** Alessandro  
**Stato:** PROPOSTE NON APPROVATE, non implementate e non distribuite ai server.

Qui posso sperimentare idee di sicurezza, isolamento e resilienza senza cambiare le decisioni di Alessandro sul suo ramo, né `main`.

## Indice

- [ARCHITETTURA.md](ARCHITETTURA.md) — diagrammi, confini di fiducia, ruoli, Yggdrasil, MCP, broker Zinga e canali opzionali.
- [DIFESA.md](DIFESA.md) — difesa a strati, hardening Linux e componenti nftables/Fail2Ban/Falco/Apache2/SSH/Tor/Cloudflare.
- [RIPRISTINO.md](RIPRISTINO.md) — guasti, backup, blocchi sicuri, sostituzione moduli e prove di recupero.
- [SCIAMI.md](SCIAMI.md) — proposte di agenti, permessi, sorveglianza posta/Telegram e test Onion.
- [ROADMAP.md](ROADMAP.md) — fasi progettuali e criteri di accettazione.
- [modulo.template.yaml](modulo.template.yaml) — modello descrittivo per aggiungere moduli senza introdurre credenziali nel Git.

## Vincoli non negoziabili ricevuti da Alessandro

1. **Solo Lia** coordina logicamente gli sciami tramite ChatGPT; i demoni Linux eseguono compiti specifici e monitoraggio, non prendono il ruolo di coordinatori autonomi.
2. **ALE** ospita endpoint **MCP** e database **PostgreSQL** (servizi separati).
3. **ZINGA** è il broker obbligatorio per le operazioni applicative e il custode delle chiavi; nessun modulo deve bypassarlo.
4. **RAGNO** (8 GB RAM, CPU esatta da verificare) ospita il server mail, scritto **interamente in Rust**.
5. **ALEX** ospita **Apache2**; **AAA** (mini PC Ryzen 5) ha ruolo **ancora da assegnare**.
6. Tutte le macchine usano **Yggdrasil**; la topologia modulare definitiva non è ancora progettata da Alessandro.
7. La posta deve poter essere letta su richiesta in ChatGPT tramite MCP. Un monitor sempre attivo potrà notificare su Telegram l'arrivo di mail attese; non implica attività di Lia fuori dalla chat.
8. **OpenStack, ZeroTier e Tor/Onion** sono ingredienti disponibili o valutabili; non significa che ogni richiesta debba attraversarli tutti.
9. La cartella contiene *idee*, non istruzioni di rollout in produzione. Ogni cambio reale richiede verifica e autorizzazione di Alessandro.

## Il compromesso di progetto

**Difesa profonda, superficie minima, riparazioni rapide.** Evitiamo strati finti che rendono impossibile capire perché qualcosa non funziona. Ogni controllo deve avere responsabilità, test, diagnostica e ripristino.

**Repository pubblico:** nessun token, secret, chiave privata, numero di telefono, IP privato reale, ID di rete, routing di produzione o dati di clienti.

**Fonti tecniche di riferimento** (da verificare prima della messa in produzione):
- [Yggdrasil — sicurezza](https://yggdrasil-network.github.io/implementation.html)
- [Cloudflare Tunnel](https://developers.cloudflare.com/tunnel/concepts/)
- [Cloudflare — DNS e posta](https://developers.cloudflare.com/dns/troubleshooting/email-issues/)
- [Falco — documentazione](https://falco.org/docs/)
- [Fail2Ban — azione nftables](https://github.com/fail2ban/fail2ban/blob/master/config/action.d/nftables.conf)
- [ZeroTier — rules engine](https://docs.zerotier.com/rules/)
- [OpenStack — requisiti di installazione](https://docs.openstack.org/install-guide/environment.html)
