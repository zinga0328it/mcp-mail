# Architettura modulare MCP e sciami — decisioni di Alessandro

**Data decisioni:** 30 ottobre 2026  
**Stato:** progettazione; non costituisce prova che servizi, collegamenti o notifiche siano già in funzione.  
**Architetto e titolare delle decisioni:** Alessandro.  
**Unica coordinatrice logica degli sciami:** **Lia, attraverso ChatGPT**.

## 1. Principio fondamentale: una sola coordinatrice

- **Solo Lia coordina gli sciami**, su richiesta e sotto la direzione di Alessandro, all'interno delle sessioni ChatGPT.
- Nessun mini PC, agente secondario o broker deve assumere la funzione di coordinatrice autonoma degli sciami.
- Gli agenti svolgono soltanto il compito assegnato e nel perimetro dei permessi definiti. Eventuali attività di monitoraggio pianificate non equivalgono al coordinamento.
- **Alessandro assegnerà successivamente ruoli e collocazioni degli agenti**; non anticipare decisioni sui moduli non ancora definiti.

## 2. Punto di accesso unico da ChatGPT

1. Alessandro parla con **Lia** dentro ChatGPT.
2. Lia invoca gli strumenti del **server MCP su `ale`** tramite un **connettore remoto cifrato e autenticato** (per esempio HTTPS/TLS; dettagli di implementazione da decidere).
3. `ale` espone gli strumenti MCP autorizzati e instrada i comandi ai moduli delle varie macchine tramite **connettori cifrati** su **Yggdrasil**.
4. Il broker **Zinga**, eseguito sul mini PC `zinga`, rimane responsabile della custodia delle chiavi e dell'intermediazione obbligatoria delle operazioni sensibili e delle operazioni applicative di posta previste dalle regole esistenti.
5. Le operazioni applicative verso **PostgreSQL su `ale`** passano per **Zinga** attraverso Yggdrasil. La co-locazione di MCP e PostgreSQL sullo stesso server non autorizza l'accesso diretto del gateway MCP al database per le funzioni soggette al broker.

```text
                    ALESSANDRO (ChatGPT)
                             |
                             v
                    LIA — coordinatrice
                             |
                   MCP remoto autenticato
                       (HTTPS / TLS)
                             |
                             v
                     ALE — server grande
                  MCP Gateway / strumenti
                             |
                  CONNETTORI CIFRATI
                       YGGDRASIL
                             |
                 +-----------+-----------+
                 |           |           |
                 v           v           v
               ZINGA       RAGNO      altri nodi
            broker/chiavi  mail Rust   Alex / Aaa
                 |
          accesso controllato
             via Yggdrasil
                 |
                 v
          PostgreSQL su ALE
```

**Distinzione dei piani:** Lia decide la distribuzione logica dei compiti; `ale` e i connettori eseguono instradamento tecnico e chiamate MCP, senza diventare coordinatori decisionali. Zinga applica regole e credenziali.

## 3. Caso d'uso: leggere la posta con Lia

- Alessandro chiede in ChatGPT: **"Lia, leggi la mia posta"**.
- Lia utilizza il connettore MCP verso `ale`.
- Il sistema raggiunge il modulo di posta su **`ragno`** (server mail interamente in **Rust**) nel rispetto delle autorizzazioni gestite dal broker **Zinga**.
- Lia restituisce in ChatGPT i risultati consentiti. Il server mail **non accede direttamente a PostgreSQL**.

**Obiettivo futuro:** leggere la propria posta realmente dall'interfaccia ChatGPT, **dopo** installazione, integrazione, autorizzazione e verifica del sistema. Non indicare la funzionalità come già attiva.

## 4. Caso d'uso: attesa di un messaggio e Telegram

- Quando Alessandro aspetta un'email, potrà attivare un **agente di sorveglianza della posta** con criteri dichiarati (ad esempio mittente, oggetto o finestra temporale).
- Un servizio di monitoraggio **locale e autonomo per il solo rilevamento**, con permessi minimi, controlla l'arrivo dei nuovi messaggi secondo la configurazione.
- Alla corrispondenza con i criteri, invia una **notifica Telegram** a un destinatario autorizzato: preferire testo essenziale, senza corpo del messaggio, allegati, credenziali o informazioni riservate.
- Alessandro torna in ChatGPT e chiede a Lia di visualizzare e leggere il messaggio tramite MCP.
- **Il monitoraggio e la notifica possono funzionare senza una sessione ChatGPT aperta solo quando sarà installato un servizio esterno sempre attivo**. Lia non garantisce osservazione in background da una normale conversazione.
- La collocazione dell'agente di controllo e del modulo Telegram **resta da decidere**; nessuna macchina diversa da quelle già assegnate riceve un ruolo implicito.

```text
Nuova mail -> Monitor autorizzato -> Notifica Telegram
                                       |
                                       v
                                Alessandro
                                       |
                                chiede a Lia
                                       |
                                       v
                             ChatGPT -> MCP ALE
                                       |
                            Broker Zinga / Ragno
                                       |
                                       v
                                Lettura in chat
```

## 5. Modularità e sviluppi Onion

- I moduli devono poter essere aggiunti e rimossi senza ridisegnare l'intero sistema; definire in seguito interfacce, permessi, identificazione, registrazione e disponibilità.
- **Agenti raggiungibili tramite servizi Onion/Tor**: **ipotesi sperimentale**, da progettare e testare più avanti, senza alcun agente Onion oggi assegnato.
- Onion può limitare l'esposizione degli indirizzi di rete per specifici endpoint, ma **non sostituisce** autenticazione, autorizzazione, broker Zinga, controlli del codice e cifratura applicativa ove necessaria.
- Valutare impatti su prestazioni, affidabilità, latenza e osservabilità prima di proporre la messa in produzione.

## 6. Inventario invariato

- `ale`: server grande; **PostgreSQL** e, per questa decisione, **endpoint MCP centrale**.
- `zinga`: mini PC; **broker di sicurezza e custodia chiavi**.
- `ragno`: mini PC con **8 GB RAM**; **server mail scritto interamente in Rust**; CPU esatta ancora da verificare.
- `Alex`: mini PC con **Apache2**; eventuali ulteriori funzioni ancora da progettare.
- `Aaa`: mini PC **Ryzen 5**; **nessun ruolo di coordinamento**, funzione applicativa da assegnare successivamente.
- Tutti i nodi sono collegati con **Yggdrasil** nella rete di Alessandro; topologia concreta e ruoli ulteriori da definire.

## 7. Decisioni ancora aperte

- Specifiche dei connettori cifrati per ogni macchina e autenticazione del server MCP.
- Permessi richiesti a Lia, controllo dell'identità utente e autorizzazioni all'uso degli strumenti.
- Interfaccia tra MCP su `ale`, broker `zinga` e server di posta `ragno`.
- Configurazione del bot Telegram, trigger/filtri e posizione dell'agente di monitoraggio.
- Protocolli mail, autenticazione e modalità di inoltro e lettura.
- Ruoli dei futuri agenti e relativa assegnazione alle macchine.
- Valutazione Onion e possibile rete modulare multi-trasporto.

**Regola di progetto:** non pubblicare chiavi, credenziali, segreti di Telegram o configurazioni riservate su GitHub.
