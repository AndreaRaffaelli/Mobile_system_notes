DOMANDE DIVISE PER CAPITOLO

Nelle pagine sotto trovate altre domande “sparse”

Casuali

• Android = modi possibili di scambiare msg (Handler?)

• Come realizzeresti un'applicazione per il pagamento del parcheggio (struttura e interazioni).

• Spiegare perchè muovendosi verso i confini di una ipotetica area coperta da un access point WIFI (al quale si è connessi) si perde QOS.

• ICMP V4 e V6.

1-Wifi 

• Hidden / Exposed node (come si possono limitare).

• CSMA/CA riduce Hidden.

• GSM.

• MS [Um, MT, TE] + BSS [BTS, BSC, Abis] + MSC [HLR, VLR] + OSS [AUC, EIR, OMC, NMC, ADC].

• Handoff GSM (messaggi che vengono scambiati; single MSC, multi MSC; varie tipologie di handoff; perché preallocazione risorse; perché MSC Anchor). 

• MSC Anchor → perché lì ci tengo delle informazioni sulla chiamata, soprattutto quelle per il charging, che non posso o sono troppo "scomode" da trasferire ad ogni cambio di MSC. Net initiated.

• Bluetooth (protocollo inquiry-page per creazione piconet; comunicazione time-slices; comunicazione non diretta tra slave).

2-MANET

• Manet 

• DSR (fault nei casi con e senza cache sui nodi; quali informazioni ha senso salvare sui nodi se si vuole fare cache).

• propagazione di errori: default-> se un nodo non riesce a trasmettere invia un messaggio RERR indietro verso il nodo sorgente (come?-> source routing, nell'header è specificato l'intero percorso, segue il percorso inverso). Il nodo sorgente fa una nuova discovery (di nuovo flooding di RREQ). Se c'è caching, la propagazione di RERR invalida le cache, può essere bloccata da un nodo che ha un path alternativo. Le migliorie che si possono introdurre sono: utilizzo delle cache sui nodi intermedi per ridurre la latenza; store di pacchetti sul nodo A; una volta ristabilito il path bisogna fare state transfer verso un nodo del nuovo path; riciclaggio della parte di path rimasta integra, ovvero quella tra S e A e riavvio del protocollo da parte di A.

• flooding (problemi di collisione): Introducendo dei TTL ai messaggi; Imponendo ai nodi di non fare forward di messaggi già ricevuti; Attesa di ritrasmissione random in caso di collisioni.

• AoDV (failure detection; timeout per RREQ/RREP; tabelle sui nodi).

• differenza tra i timeout del link avanti e dei link indietro e a chi vengono mandati i messaggi di RERR (solo ai nodi "attivi", e lui "ok, attivi in base a quale timeout?". Risposta: quello dei link avanti, configurati dal passaggio di RREP).

• GPSR (routing Geografico).

• Dense manet.

3-Mobile IP & Posizione

• Mobile IP (2 tipi di CoA; handoff tra FN; triangolar/quadrilateral routing; MIPv6 & HMIPv6).

• Handoff FN = client initizted, orizzontale, reattivo. Va spiegato che il NM deve ottenere un nuovo COA dal nuovo FA e deve aggiornare la entry nella mobility binding list del HA (eventualmente anche del CN se in MIPv6perchè trinagular di default). Il NM resta disconnesso finchè non ottiene un COA e lo aggiorna presso l'HA, fino ad allora HA continua ad inoltrare i messaggi verso il vecchio COA.

• GPS (algoritmi di triangolazione; D-GPS).

• RADAR (vs Ekahau).

• Android (Architettura; Threading+Sicurezza; Intent; BroadcastReceiver; Binder IPC)

• Verifica del bytecode come in J2ME? Viene fatta anche in Android al momento della pubblicazione dell'applicazione sullo store ma è più lasca in quanto il modello di programmazione e la suddivisione "1 DVM - 1 thread - 1 app" impongono già dei vincoli di sicurezza.

• Cosa succede quando viene lanciato un intent (viene avviata una nuova activity, portata in cima allo stack della conversazione corrente), cosa succede all'activity che lo lancia (va in stato paused), nel caso di più activity con intent filter compatibile quale viene lanciata (se c'è, activity di default, altrimenti scelta dell'utente). In caso volessimo far partire più activity? uso sendBroadcast() invece di startActivity() e definisco BroadcastReceiver con intent filter compatibile.

• Service e AsyncTask: differenze (i service non hanno un thread dedicato, gli AsyncTask sì).

5-MW & Pattern

• Pattern distribuzione (remote proxy); gestione risorse; comunicazione (Client Initiated connection).

• Proxy: es. mail in push per app mobili cellular.

• Pattern Connection Factory.

6-Discovery & Session

• JINI (broker+client+provider; lease)

• LEASE: vantaggi rispetto a deregistrazione manuale (affidabilità, il provider se cade non riuscirebbe a deregistrarsi, e minore impegno di banda per i messaggi scambiati). Il lease è negoziato tra provider e broker; il provider propone e il broker decide se accettare o meno in base alle proprie politiche interne.

• JSPO può avere un timeout interno che alla scadenza del lease non consente di effettuare più richieste. JSPO scritto da provider. 

• FEDERAZ: (broker si registra come provider presso un altro broker) 

• REPLICHE: (1)→registrazione a tutti. Se qualcosa si rompe non avviso nessuno. (2)→ registro a uno e lui propaga. Ma se si rompe il primo devo avvisare provider di registrarsi all’altro, meglio la precedente. 

• UPnP (discovery, eventi, GENA, quando multicast/unicast)

• DISCOVERY = due possibilità (ssdp:discover, ssdp:alive).

• MSG = tra device e control point; per ogni messaggio quale protocollo si usa, quali sono fatti in multicast. GENA ha voluto sapere cosa invia in risposta il device al control point (solo cambiamenti). La Notify è fatta in unicast. 

• SIP (protocollo; esempio d'uso).

7-Eventi

• Pub/Sub (routing sottoscrizioni [simple, id based, covering, merging]).

  
  
  

ALTRE DOMANDE

  

- threading in Android
    
- Handoff tra due FA in mobile IP + idea HMIP
    
- Risolviamo con MACA tutti i problemi?
    
- Definizione e mini analisi bridging UPnP
    

- MIP e PMIP

- Merging based routing

  

Intent (cosa sono, espliciti e impliciti, come lanciarli con anche un esempio di codice)

broadcast receiver.

  

differenza mip pmip e cosa succede durante handoff, quanto è lungo delta t disservizio, ekahu radar differenze, differenza tra ctivity service e async task con codice, hidden node + maca

  

Upnp, sottoscrizione eventi : cosa sono  variabili di stato? a cosa serve il timeout nella sottoscrizione ? (Leasing, stesso meccanismo di jini)

  

Mobile IP, come avviene l’handoof? Quando il nodo esce dalla Foreign network 1 avvisa? NO Come è strutturato PMiPv6? componenti principali?

  

Android, AsyncTask con codice e differenze rispetto service e thread-- GUI interaction

  

Bluetooth, collisioni? No by design ma se un nodo arriva  e comincia a comunicare potrebbe succedere, stile MACA. SCO vs ACL ?

  
  
  

Domande 27/07/2016 mattinata 

  

XMPP

D-GPS

AODV

MidLet Sicurezza - confronto Android

MobileIP - Handoff

HMip VS MobileIPv6 

MACA

Eventi UPNP GENA 

Thread e AsyncTask - codice

Radar VS Ekahu

Precisione VS Accuratezza

Errori GPS 

Ottimizzazione Flooding 

Bluetooth - differenza fra SCO e ACL

Activity vs service vs AsyncTask

Differenza Service e AsyncTask per interazione UI 

GPSR 

DSR cosa succede per link failure

UPNP - invocazione CP - Device

Android threading 

  

Domande 05/07/19

  

AODV -> formazioni routing table e gestione di RERR

Android -> Intent e BroadcastReceiver

ZigBee -> topologie e modalità di comunicazione

I-TCP -> perché si utilizza, come funziona e la violazione del principio end-to-end

  

Domande 29/02/21

AODV -> formazioni routing table e gestione di RERR

Mobile ip -> come avviene l’handoff tra un foreign agent e un altro e se vengono persi pacchetti

JINI (apache river) -> più o meno tutto, in particolare: “c’è qualche sistema per il discovery del jini broker stesso?: no, i client devono conoscere la locazione del jini broker”

  

Domande 16/06/2022

Candidato 1 voto 20

1.     Comunicazioni wireless

a.     Si parli dei problemi dell’hidden node e exposed node

b.     Maca per WiFi risolve questi problemi?

c.      Caratteristiche del protocollo MACA

d.     Per quanto tempo i nodi stanno in silenzio?

e.     Il problema dell’exposed node è risolto da MACA?

f.      Come può in nodo mobile interagire con questo protocollo?

2.     Android

a.     Come funzionano gli intent in android?

b.     Come può essere mandato un intent? Unicast multicast broadcast

c.      Quali sono le api relative all’intent

d.     È android che fa match degli intent con l’activity

e.     Come avviene il match tra intent e l’activity?

f.      Cosa succede se ci sono più app che matchano l’intent?

g.     Come si registra il braodcast receiver presso gli intent a cui è interessato?

3.     Mobile Ip

a.     Cosa succede nell’handoff in mobile Ip

b.     Cosa succede quando l’MH si sposta in una località diversa

c.      C’è perdita di pacchetti?

d.     Durante lo spostamento in una località  i pacchetti continuano ad arrivare all’fa?

e.     Il vecchio fa effettua la ritrasmissione dei pacchetti ?

f.      Mobile ip è reattivo o proattivo?

g.     È verticale o orizzontale?

4.     Differenze tra SCO e ACL in bluetooth

Candidato 2 voto 30

1.     MANET AODV

a.     Come sono fatte le tabelle ?

b.     Come viene gestito il link fault?

c.      Cosa succede in caso di loop?

d.     Cosa succede se nessuno ha un path verso la d dopo il fault?

e.     Cosa si intende per protocollo ibrido?

2.     Android

a.     Si parli degli asynctask

b.     Come vengono gestiti i thread?

c.      Lo sviluppatore deve estendere AsyncTask?

d.     I thread devono essere istanziati?

e.     Cosa comporterebbe non avere gli asynctask?

3.      GSM wireless communication

a.     Come avviene la gestion dell’handoff in GSM?

b.     Reattivo o proattivi?

c.      Ci possono essere perdite di pacchetti?

d.     Durante una chiamata lunga si crea una catena?

4.     Service discovery upnp

a.     Parlare del supporto alle notifiche

b.     Se ci sono molti control point registrati per lo stesso evento come viene gestita la propagazione? (No ottimizzato)

Candidato 3  voto 30 esame con attività progettuale su iot con uso di mqtt

1.     Internet of things

a.     Mqtt quali sono le semantiche dei messaggi?

b.     Vantaggi dell’applicare la crittografie al payload del messaggio o a tutto il messaggio

c.      Con payload encription si può cifrare tutto il payload o anche solo parti?

d.     Ci sono modi per non asofraccaricare il broker con la decription nel caso in cui sia cifrato l’intero messaggio?

2.     Gestione degli eventi

a.     Modalità di propagazione delle Subscription in convering based routing?

b.     Come si risolve il problema della cancellazione delle sottoscrizioni più generali?

3.     Manet gpsr

a.     Parlare di gpsr

b.     Greedy forwarding

c.      Perimeter forwarding

d.     Come viene costruito il grafo per il perimeter forwarding?

4.     Sistemi di posizionamento

a.     Differential GPS

b.     Differenza tra precisione e accuratezza

c.      Perché non si risolve il multipath fading?

Candidato 4 voto 25

1.     Wireless communication bluetooth

a.     Differenze tra SCO e ACL

2.     Mobile ip

a.     Differenze tra ipv4 e ipv6

b.     Problemi scarto pacchetti ingresso e uscita?

c.      Il problema di avere lo stesso indirizzo c’è anche in ip v4?

d.     Perché mipv6 non è usato ?

e.     In mipv4 si possono perdere pacchetti succede anche in mipv6 con il tunneling?

3.     Android

a.     Multithreading in android

b.     Differenze tra service bound o unbounded?

c.      Api per definire bound e unbound services?

d.     Dal punto di vista dei thread come si gestiscono i service?

e.     Se sono dentro la stessa app come sono assegnati i thread?

4.     Sistemi di posizionamento

a.     Differenza tra KAU e radar

Candidato 5 voto 30L

1.     Android

a.     Asyntask

b.     Primitive di asynctask

c.      Come sono suddivisi i thread

d.     Quali alternative sono possibili all’uso di asynctask

e.     Sicurezza in android

f.      Come fa un app che non contiene al suo interno un file ad accedere al file di un'altra app? Condivide gid

g.     Come si accede al file system in android?

2.     Wireless communication

a.     GSM come viene gestito l’handoff nella stessa località?

b.     Come viene gestito tra località diverse?

c.      Viene mantenuta la catena di msc?

d.     Può esservi la perdita di pacchetti nell’handoff?

3.     Wireless communication

a.     Zigbee quali sono le topologie?

b.     Quali sono i ruoli?

4.     Mobile ip

a.     Differenze tra mipv4 e mipv6?

b.     Ottimizzazioni del triangular routing?

c.      Problema di mipv6 perché è poco adottato?

d.     Ottimizzazioni presenti?

5.     MANET

a.      Come funziona il flooding ?

b.     Quando viene utilizzato?

c.      Come si può ottimizzare?

d.     Broadcast storm issue?