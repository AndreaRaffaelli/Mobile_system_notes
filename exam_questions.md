# Domande d'Esame \[AI generated\]- Sistemi mobili

## 1. ANDROID

### Architettura e Threading
- Descrivi l'architettura a livelli di Android e spiega il ruolo del HAL
- Quali sono le differenze tra DVM e ART? Perché è stato fatto il passaggio?
- Come funziona il modello single-thread in Android e quando è necessario utilizzare thread aggiuntivi?
- Differenze tra Thread, Handler, AsyncTask e Service per operazioni asincrone
- Come può un thread secondario comunicare con l'UI thread? Mostra un esempio di Handler
- Cosa succede se provi a modificare la UI da un thread diverso dal main thread?

### Activity e Lifecycle
- Descrivi il ciclo di vita di un'Activity con tutti i callback
- Cosa succede quando un'Activity passa da RUNNING a PAUSED? E da PAUSED a STOPPED?
- Come funziona lo stack LIFO delle Activity in un Task?
- Cosa contiene il Bundle e quando viene utilizzato?

### Intent e Comunicazione
- Differenze tra Intent espliciti e impliciti con esempi di codice
- Come funziona l'Intent resolution algorithm?
- Cosa succede se più Activity hanno Intent Filter compatibili?
- Come si implementa e registra un BroadcastReceiver (staticamente e dinamicamente)?
- Differenze tra sendBroadcast(), sendOrderedBroadcast() e LocalBroadcastManager
- Come funziona il Binder IPC e perché è importante per la sicurezza?

### Services e Background Processing
- Differenze tra Unbound Service, Bound Service e Foreground Service
- Quando usare IntentService invece di Service normale?
- Come funziona JobScheduler e perché è più efficiente di AlarmManager?
- Mostra il codice per implementare un AsyncTask con tutti i callback

### Sicurezza
- Come funziona il sistema di permessi basato su UID/GID?
- Cosa sono i WakeLocks e come rappresentano il pattern Cross-layering?
- Come può un'app accedere ai file di un'altra app?

## 2. RETI WIRELESS

### WiFi e Protocolli di Accesso
- Problemi di Hidden Node e Exposed Node: definizione e soluzioni
- Come funziona CSMA/CA e perché è necessario in wireless?
- MACA risolve completamente i problemi di Hidden ed Exposed Node?
- Per quanto tempo i nodi restano in silenzio in MACA dopo aver sentito RTS/CTS?
- Come gestisce MACA l'arrivo di un nodo mobile durante una comunicazione?

### GSM
- Descrivi l'architettura GSM: MS, BSS, MSC, OSS e loro componenti
- Come avviene l'handoff in GSM (stesso MSC vs diversi MSC)?
- Perché si usa la preallocazione delle risorse nell'handoff GSM?
- Cosa è l'MSC Anchor e perché è necessario?
- L'handoff GSM è reattivo o proattivo? Ci sono perdite di pacchetti?

### Bluetooth
- Differenze tra SCO e ACL in Bluetooth
- Come funziona il protocollo inquiry-page per la creazione di piconet?
- Come comunicano due slave che non possono parlare direttamente?
- Ci sono problemi di collisione in Bluetooth? Come vengono gestiti?

### ZigBee
- Quali sono le topologie supportate da ZigBee?
- Differenze tra Coordinator, Router ed End Device
- Come avviene la comunicazione in una topologia mesh?

## 3. MANET

### Routing Proattivo vs Reattivo
- Differenze fondamentali tra protocolli proattivi e reattivi
- Vantaggi e svantaggi di ciascun approccio
- Quando conviene usare l'uno o l'altro?

### DSR (Dynamic Source Routing)
- Come funziona il Source Routing in DSR?
- Cosa contengono i messaggi RREQ e RREP?
- Come viene gestito il link failure con e senza cache?
- Come si propaga un messaggio RERR verso la sorgente?
- Quali ottimizzazioni introduce la cache sui nodi intermedi?
- Cosa succede se un nodo ha un path alternativo quando riceve RERR?

### AODV (Ad-hoc On-demand Distance Vector)
- Che informazioni contengono le routing table in AODV?
- Differenze tra timeout per link avanti e link indietro
- Come viene gestito il link failure e la propagazione di RERR?
- A quali nodi vengono inviati i messaggi RERR e perché?
- Come si prevengono i loop in AODV?
- Cosa succede se nessun nodo ha un path verso la destinazione dopo un fault?

### GPSR (Greedy Perimeter Stateless Routing)
- Come funziona il Greedy Forwarding?
- Quando si passa al Perimeter Forwarding?
- Come viene costruito il grafo planare per il perimeter forwarding?
- Vantaggi e limitazioni del routing geografico

### Flooding e Ottimizzazioni
- Problemi del flooding semplice (broadcast storm)
- Come si può ottimizzare il flooding?
- Ruolo del TTL e della cache dei messaggi già ricevuti
- Attesa randomica per ridurre collisioni

## 4. MOBILE IP E MOBILITÀ

### Mobile IP v4
- Ruoli di Home Agent, Foreign Agent e Mobile Host
- Differenze tra FA COA e Co-located COA
- Come avviene l'handoff tra due Foreign Agent?
- Quanto dura il disservizio durante l'handoff?
- Problemi del triangular routing
- Problemi con firewall (entering/exiting filter)

### Mobile IP v6
- Come MIPv6 risolve il problema del triangular routing?
- Cosa sono i binding updates?
- Perché MIPv6 non è ampiamente adottato?
- Problemi di compatibilità e deployment

### HMIPv6 (Hierarchical Mobile IPv6)
- Come funziona la gerarchia di Foreign Agent?
- Differenze tra local subnet handoff e MAP domain handoff?
- Vantaggi in termini di binding updates

### Proxy Mobile IP (PMIPv6)
- Differenze tra MIP e PMIP
- Ruoli di LMA (Local Mobility Anchor) e MAG (Mobile Access Gateway)
- Vantaggi del non coinvolgere il mobile host

### I-TCP
- Perché si utilizza I-TCP invece di TCP standard?
- Come I-TCP gestisce la mobilità e gli handoff?
- Violazione del principio End-to-End: conseguenze
- Integrazione con Mobile IP

## 5. SISTEMI DI POSIZIONAMENTO

### Concetti Base
- Differenza tra precisione e accuratezza
- Sistemi fisici vs logici, assoluti vs relativi
- Tecniche di lateration: ToA, RSSI, TDoA, Angulation

### GPS
- Come funziona la triangolazione GPS?
- Principali fonti di errore in GPS
- Perché serve un quarto satellite?
- Come funziona D-GPS e che miglioramenti apporta?
- Perché D-GPS non risolve il multipath fading?

### Sistemi Indoor
- Differenze tra RADAR e Ekahau
- Come funziona il WiFi fingerprinting?
- Vantaggi e svantaggi di Active Badge vs Active BAT
- Scene Analysis: preparazione e utilizzo

### Positioning in MANET
- Come funziona ABC (Assumption Based Coordinates)?
- Ruolo dell'Anchor Node
- Estensione Terrain per copertura multi-hop

## 6. DISCOVERY E SESSION MANAGEMENT

### Jini/Apache River
- Componenti: Service, Client, Discovery Service
- Come funziona il meccanismo di lease?
- Vantaggi del lease rispetto alla deregistrazione manuale
- Java Service Proxy Object: vantaggi
- Come funziona la federazione tra broker?
- Gestione delle repliche: strategie

### UPnP
- Fasi del protocollo UPnP: discovery, description, control, eventing
- SSDP: quando si usa multicast vs unicast?
- Differenza tra ssdp:discover e ssdp:alive
- Come funziona la sottoscrizione agli eventi con GENA?
- Cosa invia il device al control point nelle notifiche?
- UPnP bridging: problemi e soluzioni

### SIP (Session Initiation Protocol)
- Ruolo di SIP nel setup delle sessioni
- Componenti principali e messaggi
- Esempio d'uso pratico

## 7. IoT e MESSAGING

### Protocolli IoT
- Classificazione dispositivi IoT (Classe 0, 1, 2)
- Differenze tra CoAP e HTTP
- Livelli di affidabilità in CoAP (CON, NON, ACK)
- Come funziona 6LoWPAN?

### MQTT
- Semantiche dei messaggi: at most once, at least once, exactly once
- Come si implementa "exactly once" con 4 messaggi?
- Retained Messages e Will: quando usarli?
- Vantaggi della crittografia del payload vs messaggio completo

### Confronto Protocolli
- MQTT vs AMQP vs DDS: quando usare quale?
- Pub/Sub vs Request/Response: vantaggi e svantaggi

## 8. EVENTI E PUBLISH/SUBSCRIBE

### Architetture Event Router
- Centralized vs Hierarchical vs Peer-to-Peer
- Rendezvous Point: vantaggi

### Routing delle Sottoscrizioni
- Simple flooding vs Covering-based vs Merging-based
- Come si risolve il problema della cancellazione delle sottoscrizioni generali nel covering-based?
- Problemi dei falsi positivi nel merging-based

### Sistemi Specifici
- Java Model for Distributed Events: handback object
- GENA in UPnP: timeout nelle sottoscrizioni
- DDS: partizioni e QoS

## 9. MIDDLEWARE E PATTERN

### Pattern di Distribuzione
- Remote Proxy: quando e come usarlo
- Data Transfer Object: vantaggi in mobile
- Facade Pattern: esempio di implementazione

### Pattern di Gestione Risorse
- Session Tokens vs Stateless
- Caching: Eager vs Lazy Acquisition
- Sincronizzazione: Pessimistic vs Optimistic

### Pattern di Comunicazione
- Connection Factory: vantaggi
- Multiplexed Connection: quando usare
- Client Initiated Connection for Push: esempi pratici

## 10. SICUREZZA E QoS

### Sicurezza Mobile
- Problemi specifici della mobilità
- Autenticazione in movimento
- Crittografia: session level vs message level

### QoS in Mobilità
- Gestione della QoS durante handoff
- Adattamento alle condizioni di rete
- Cross-layering per ottimizzazione energetica

## DOMANDE INTEGRATE

### Scenario Design
- Come realizzeresti un'app per il pagamento del parcheggio? (architettura, protocolli, gestione mobilità)
- Progetta un sistema di notifica push per app mobili considerando: protocolli, middleware, gestione eventi
- Sistema di localizzazione indoor per un ospedale: sensori, protocolli, architettura

### Analisi Comparativa
- Confronta le strategie di handoff in GSM, WiFi e Mobile IP
- MANET: quando preferire DSR vs AODV vs GPSR?
- Android vs iOS: differenze architetturali per lo sviluppo mobile

### Problem Solving
- Un utente si muove verso i confini di un AP WiFi: perché si perde QoS e come mitigare?
- Ottimizzazione energetica in una WSN: strategie a livello applicativo e middleware
- Gestione della disconnessione temporanea in un'app mobile: pattern e strategie