# WSTG-CONF-01

# Test Network Infrastructure Configuration

## Summary

La complessità intrinseca di un'infrastruttura di un web server interconnessa ed eterogenea, che può includere centinaia di web app, rende la gestione e la verifica delle configurazioni un passo fondamentale nel testing e nel deploy di ogni singola app.
Basta una singola vulnerabilità per compromettere la sicurezza di tutta l'infrastruttura, e anche problemi piccoli e di poca importanza possono trasformarsi in gravi rischi per altre app sullo stesso server.
Per risolvere questi problemi, è di massima importanza eseguire una verifica approfondita delle configurazioni e delle issue di sicurezza conosciute, dopo aver mappato l'intera architettura.

È importante un'adeguata gestione delle configurazioni dell'infrastruttura del web server per preservare la sicurezza della stessa app.
Se elementi come il software del web server, i database server del backend, o l'authorization server non sono verificati e resi sicuri adeguatamente, potrebbero introdurre richi indesiderati o nuove vulnerabilità che potrebbero compromettere la stessa app.

Per esempio, 
una vulnerabilità del web server che potrebbe consentire a un attaccante remoto di recuperare il codice sorgente della stessa app (una vulnerabilità che è sorta più volte sia sul web server che sul server applicativo) 
potrebbe compromettere l'app, dato che utenti anonimi potrebbero usare le informazioni divulgate del codice sorgente per implementare un attacco contro l'app o contro i suoi utenti.

I seguenti passi devono essere eseguiti per verificare l'infrastruttura della gestione delle configurazioni:

- i diversi elementi che compongono l'infrastrutura devono essere determinati per capire come interagiscono con una web app e come impattano sulla sua sicurezza
- tutti gli elementi dell'infrastruttura devono essere verificati per assicurarsi che non contengano vulnerabilità conosciute
- bisogna eseguire una verifica sui tool amministrativi utilizzati per la manutenzione dei diversi elementi
- i sistemi di autenticazione devono essere verificati per assicurarsi che soddisfino i requisiti dell'app e che non possano essere sfruttati da utenti esterni per eseguire un qualsiasi accesso
- bisogna mantenere aggioranta e sotto controllo una lista di porte che sono necessarie all'app

Dopo aver mappato i diversi elementi che compongono l'infrastruttura è possibile verificare la configurazione di ogni elemento trovato e testarla per eventuali vulnerabilità conosciute.

## Test Objectives

Mappare l'infrastruttura che supporta l'app e capire come impatta sulla sua sicurezza.

## How to Test

### Known Server Vulnerabilities

Le vulnerabilità trovate nelle diverse aree dell'architettura dell'app, siano esse nel web server o nel database di backend, possono compromettere la stessa app.
Per esempio, considera una vulnerabilità del server che consente a un utente remoto non autenticato di caricare file sul web server o perfino di sovrascriverli.
Questa vulnerabilità potrebbe compromettere l'app, dato che un utente malintenzionato potrebbe rimpiazzare la stessa app o introdurre codice che potrebbe impattare sui server di backend, dato che il suo codice verrebbe eseguito come una qualsiasi altra app.

La verifica delle vulnerabilità del server può essere difficile da fare se il test viene eseguito secondo un blind penetration test.
In questi casi, le vulnerabilità devono essere testate da un sito remoto, di solito usando un tool automatico.
Comunque, il testing di alcune vulnerabilità può avere dei risultati non predicibili sul web server, e il testing di altre vulnerabilità (come i DoS) potrebbe non essere accettabile per un eventuale downtime di servizio in caso di test avvenuto con successo.

Alcuni tool automatici notificano le vulnerabilità in base alla versione del web server recuperata.
Ciò porta sia a falsi positivi che a falsi negativi.
Da un lato, se la versione del web server è stata rimossa oppure offuscata dall'amministratore il tool di scan non è in grado di segnalare il server come vulnerabile anche se lo è.
Dall'altro lato, se il vendor che fornisce il software non aggiorna la versione del web server quando le vulnerabilità sono state fixed, il tool di scan segnala vulnerabilità che non esistono.
Il secondo caso è molto comune dato che alcuni vendor di SO forniscono le patch per vulnerabilità di sicurezza, 
ma non eseguono un passaggio completo all'ultima versione del software.
Questo succede principalmente in molte distribuzioni come Debian, Red Hat o SuSE.
Nella maggior parte dei casi, lo scanning di vulnerabilità su un'architettura di un'app troverà solo vulnerabilità associate  agli elementi "esposti" dell'architettura (come il web server) 
e sarà di solito incapace di trovare vulnerabilità associate agli elementi che non sono esposti direttamente, come il back end di autenticazione, il database di backend, o il reverse proxy in uso.

Infine, non tutti i vendor di software divulgano le vulnerabilità in modo pubblico, 
e quindi queste debolezze non vengono registrate nei database pubblici di vulnerabilità conosciute.
Queste informazioni sono solo divulgate ai clienti o pubblicate tramite fix che non sono accompagnate da alcun advisory.
Ciò riduce l'utilità dei tool di scanning di vulnerabilità.
Di solito, la vulnerability coverage di questi tool sarà molto buona per prodotto comuni (come web server Apache, Internet Information Server (IIS) di Microsoft, o Lotus Domino di IBM) ma sarà carente per i prodotti meno conosciuti.

Per questo motivo è meglio eseguire la verifica delle vulnerabilità quando si hanno informazioni interne del software usato, incluse le versioni e le release usate e le patch applicate al software.
Con queste informazioni, il tester può recuperare le informazioni dallo stesso vendor e analizzare quali vulnerabilità potrebbero essere presenti nell'architettura e come possono impattare sull'app stessa.
Quando possibile, queste vulnerabilità possono essere testate per determinare i loro effetti reali e rilevare se ci potrebbe essere un elemento esterno (come un instrusion detection o prevention system) che potrebbe ridurre o impedire la possibilità di un exploit di successo.
I tester potrebbero anche determinare, tramite una verifica della configurazione, che la vulnerabilità non è presente, dato che riguarda un componente del software che non viene usato.

Vale anche la pena notare che a volte i vendor possono fare il fixing di vulnerabilità silenziosamente e rendere disponibili le fix con la nuova release software.
Diversi vendor hanno cicli di release diversi che determinano il supporto che possono dare alle release più vecchie.
Un tester che ha informazioni dettagliate sulle versioni software usate nell'architettura 
può analizzare il rischio associato all'uso di release vecchie del software che potrebbe non essere supportato nel breve periodo o potrebbe essere già non più supportato.
È un aspetto critico, dato che se una vulnerabilità riguardava una vecchia versione software che non è più supportata, 
il personale potrebbe non esserne al corrente.
Non saranno mai rese disponibili delle patch e gli advisory potrebbero non elencare questa versione dato che non è più supportata.
Anche se sono a conoscienza che la vulnerabilità è presente e il sistema è vulnerabile, 
dovranno fare un aggiornamento completo a una nuova release software, 
che potrebbe introdurre un downtime significativo nell'architettura dell'app o potrebbe forzare la riscrittura dell'app a causa di incompatibilità con l'ultima versione software.

### Administrative Tools

Qualsiasi infrastruttura di un web server ha bisogno di tool amministrativi per il mantenimento e l'aggiornamento delle informazioni usate dall'app.
Queste informazioni includono i contenuti statici (pagine web, file grafici), il codice sorgente dell'app. il database di autenticazione degli utenti, etc.
I tool amministrativi saranno differenti in base al sito, alla tecnologia, o al software usato.
Per esempio, alcuni web server vengono gestiti usando interfacce amministrative che sono, esse stesse, web server (come il web server iPlanet) 
o vengono gestiti tramite file di testo in chiaro (Apache) 
o usano tool GUI del SO (quando si usa IIS di Microsoft o ASP.Net).

In molti casi la configurazione del server viene gestita usando diversi tool di mantenimento dei file usati dal web server, che sono gestiti tramite server FTP, WebDAV, network file system (NFS, CIFS) o altri meccanismi.
È ovvio che il SO degli elementi che compongono l'architettura dell'app sono gestiti tramite altri tool.

Le app potrebbero avere anche delle interfacce amministrative incorporate usate per gestire i dati della stessa app (utenti, contenuto, etc.).

Dopo aver mappato le interfacce amministrative usate per gestire le diverse parti dell'architettura 
è importante verificarle dato che 
se un attaccante ottenesse l'accesso a una di esse
potrebbe poi compromettere o danneggiare l'architettura dell'app.
Per fare ciò è importante:

- determinare i meccanismi che controllano l'accesso a queste interfacce e la loro suscettibilità.
Questa informazione potrebbe essere disponibile online
- cambiare l'username e la password di default

Alcune aziende scelgono di non gestire tutti gli aspetti delle loro web app, ma delegano a terzi la gestione del contenuto utilizzato dalla web app.
Questa azienda esterna 
potrebbe fornire solo parti del contenuto (aggiornamenti o promozioni) o 
potrebbe gestire completamente il web server (incluso il contenuto e il codice).
È comune trovare interfacce amministrative raggiungibili da Internet in queste soluzioni, 
dato che è più economico usare Internet rispetto a fornire una linea dedicata che connette l'azienda esterna all'infrastruttura dell'app attraverso un'interfaccia di sola gestione.
In questa situazione, è molto importante testare se le interfacce amministrative sono vulnerabili agli attacchi.