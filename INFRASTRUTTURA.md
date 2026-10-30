# Infrastruttura fisica — MCP SWARM / mcp-mail

**Fonte:** indicazioni di Alessandro, architetto del progetto, 30 ottobre 2026.
**Stato:** inventario dichiarato; non equivale a una verifica remota dei sistemi.

## Macchine disponibili

| Host | Tipo / hardware noto | Ruolo stabilito | Note |
|---|---|---|---|
| **ale** | Server principale ("server grande") | Ospita il database **PostgreSQL** | Database non esposto su Internet; raggiungibile dal broker soltanto tramite rete **Yggdrasil** e con controlli di accesso |
| **zinga** | Mini PC | **Broker Zinga** e custodia/gestione delle chiavi | Punto obbligatorio di intermediazione verso il database; progettare isolamento e gestione sicura dei segreti |
| **Alex** | Mini PC | Server web **Apache2** | Altri collegamenti applicativi non ancora definiti |
| **Aaa** | Mini PC con **Ryzen 5** | Da assegnare | RAM e altri dettagli non indicati |
| **ragno** | Mini PC, **8 GB RAM**; CPU indicata come "Ryzen 3 o 4" (modello incerto) | **Server mail scritto interamente in Rust** | Modello CPU e altri dettagli da verificare |

**Attenzione ai nomi:** `ale` (server grande e database) e `Alex` (mini PC Apache2) sono due macchine distinte.

## Rete

- **Tutte e cinque le macchine** sono collegate alla rete overlay **Yggdrasil**, gestita/erogata dall'infrastruttura di Alessandro.
- La topologia reale dei nodi, gli indirizzi Yggdrasil e l'eventuale nodo di ingresso/hub **non sono ancora stati specificati**: non presumere che `ale` sia il nodo gateway.
- La sola presenza su Yggdrasil **non equivale a una policy di accesso**: prevedere autenticazione di servizio e firewall sugli endpoint.
- **PostgreSQL su ale** deve essere raggiungibile per l'applicazione **solo dal broker zinga** lungo Yggdrasil, e non deve offrire un endpoint pubblico.

## Flusso applicativo deciso

```text
   Client / protocolli mail (da specificare)
                      |
                      v
            +------------------+
            |      ragno       |
            | Mail server Rust |
            |    RAM: 8 GB     |
            +------------------+
                      |
                 Yggdrasil
                      |
                      v
            +------------------+
            |      zinga       |
            | Broker + chiavi  |
            +------------------+
                      |
                 Yggdrasil
                      |
                      v
            +------------------+
            |       ale        |
            |   PostgreSQL     |
            +------------------+

    Alex: Apache2  --> rete Yggdrasil
    Aaa: Ryzen 5   --> rete Yggdrasil (ruolo da decidere)
```

## Vincoli progettuali confermati

1. Il servizio mail su `ragno` è sviluppato **completamente in Rust**.
2. Le interazioni applicative verso PostgreSQL transitano **esclusivamente tramite il broker Zinga**: niente collegamento applicativo diretto `ragno` → `ale`.
3. `zinga` è il **signore delle chiavi**; credenziali e segreti non devono essere pubblicati nel repository.
4. `ale` ospita PostgreSQL; `Alex` ospita Apache2; `Aaa` resta disponibile per un ruolo futuro.
5. Questa è **documentazione di progetto**, non una dichiarazione di servizi già implementati o verificati.

## Punti ancora da decidere

- CPU esatta di `ragno` e capacità dei dischi.
- Ruolo applicativo di `Aaa`.
- Modalità precisa con cui Alessandro eroga/gestisce Yggdrasil e raggiungibilità dei singoli nodi.
- Protocolli mail da supportare, autenticazione di servizio, TLS e porte/servizi da esporre.
- Policy di backup e ripristino dei dati PostgreSQL.

_Non inserire qui password, chiavi private o indirizzi riservati del sistema._
