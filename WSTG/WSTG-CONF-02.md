# WSTG-CONF-02

# Test Application Platform Configuration

## Summary

È importante un'adeguata configurazione dei singoli elemeni che compongono l'architettura dell'app 
per prevenire errori che potrebbero compromettere la sicurezza dell'intera architettura.

La verifica e il testing della configurazione è un compito critico per la creazione e il mantenimento di un'architettura.
Questo perchè molti sistemi vengono forniti con configurazioni generiche che potrebbero non essere adatte ai compiti che devono svolgere nello specifico sito in cui sono installate.

Anche se l'installazione tipica di web server e app server contiene molte funzionalità (come esempi di app, documentazione, pagine di test) 
tutto ciò che non è necessario andrebbe rimosso prima del deploy per evitare un exploit dopo l'installazione.

## How to Test

### Black-Box Testing

#### Sample and Known Files and Directories

Molti web server e app server forniscono, nelle installazioni di default, app di esempio e file a vantaggio degli sviluppatori 
per verificare che il server stia lavorando correttamente subito dopo l'installazione.
Comunque, molte app di default di web server sono risultate vulnerabili.
Era il caso, per esempio,
della CVE-1999-0449 (Denial of Service in IIS quando il sito di esempio di Exair veniva installato), 
della CAN-2002-1744 (vulnerabilità di Directory traversal in CodeBrws.asp in Microsoft IIS 5.0),
della CAN-2002-1630 (uso di sendmail.jsp in Oracle 9iAS),
o della CAN-2003-1172 (Directory traversal nell'esempio di view-source in Cocoon di Apache).

Gli scanner CGI includono una lista dettagliata di esempi di file e directory conosciuti che vengono forniti da diversi web server e app server e potrebbero essere un modo veloce per determinare se questi file sono presenti.
Comunque, l'unico modo per essere veramente sicuri è fare una verifica completa dei contenuti del web server o dell'app server e determinare se sono correlati all'app o meno.

#### Comment Review

È molto comune per i programmatori aggiungere i commenti durante lo sviluppo di grandi app web-based.
Comunque, i commenti inclusi nel codice HTML potrebbero rivelare informazioni interne che non dovrebbero essere a disposizione di un attaccante.
A volte, il codice sorgente è commentato quando la funzione non è più necessaria 
ma questo commento rimane nelle pagine HTML che vengono restituite non intenzionalmente agli utenti.

La verifica dei commenti dovrebbe essere eseguita per determinare se delle informazioni vengono rivelate tramite i commenti.
Quest'analisi può essere fatta sul contenuto statico e dinamico del web server e tramite ricerca di file.
Può essere utile navigare nel sito sia in modo automatico che guidato e 
memorizzare tutto il contenuto recuperato.
Questo contenuto può poi essere analizzato per individuare eventuali commenti.

#### System Configuration

Il CIS-CAT dà ai professionisti IT e di sicurezza una valutazione veloce e dettagliata della conformità dei sistemi target ai benchmark CIS.
Il CIS fornisce anche la guida per l'hardening della configurazione del sistema inclusi i database, il SO, il web server.

### Gray-Box Testing

#### Configuration Review

Le configurazioni dei web server o app server hanno un ruolo importante nella protezione dei contenuti del sito e 
devono essere verificate attentamente per individuare errori di configurazione comuni.
È ovvio che la configurazione raccomandata varia a seconda della policy del sito, e 
a seconda delle funzionalità che dovrebbero essere fornite dal software del server.
Nella maggior parte dei casi, comunque, bisognerebbe seguire le linee guida sulla configurazone (fornita dal vendor del software o da terze parti) per verificare se il server è stato reso sicuro.

È impossibile stabilire in modo generico come un server dovrebbe essere configurato, comunque, 
bisognerebbe prendere in considerazione alcune linee guida comuni:

- abilitare solo i moduli del server che sono necessari all'app (ISAPI extension nel caso di IIS).
In questo modo si riduce la superficie d'attacco dato che viene ridotta la dimensione e la complessità del server tramite la disabilitazione dei moduli software.
La disabilitazione dei moduli previene anche le vulnerabilità che potrebbero apparire nel sofware dei moduli del vendor disabilitati
- gestire gli errori del server (40x e 50x) con pagine custom al posto di pagine di default del web server.
In particolare assicurati che qualsiasi errore dell'app non venga restituito all'utente e che 
nessun codice sia divulgato tramite questi errori dato che potrebbero aiutare un attaccante.
È molto comune dimenticarsi di questo aspetto dato che gli sviluppatori hanno bisogno di queste informazioni negli ambienti di pre-produzione
- assicurati che il software del server venga eseguito con privilegi minimi nel SO.
Ciò impedisce che un errore nel software del server comprometta l'intero sistema, anche se un attaccante potesse fare privilege escalation una volta che esegue codice sul web server
- assicurati che il software del server faccia il logging sia degli accessi legittimi che degli errori
- assicurati che il server sia configurato per gestire adeguatamente i sovraccarichi e impedisca gli attacchi DoS.
Assicurati che sia stato adeguatamente ottimizzato per le prestazioni
- non assegnare l'accesso a identità non amministrative (ad eccezione di `NT SERVICE\WMSvc`) ad applicationHost.config, redirection.config, e administration.config (accesso in Read o Write).
Ciò include `Network Service`, `IIS_IUSRS`, `IUSR`, o altre identità custom usate dai pool applicativi IIS.
Il worker process di IIS non sono stati progettati per accedere direttamente a questi file.
- non condividere mai applicationHost.config, redirection.config, e administration.config in rete.
Quando si usa Shared Configuration, è meglio esportare applicationHost.config in un'altra posizione 
(guarda la sezione "Setting Permissions for Shared Configuration")
- tieni a mente che di default tutti gli utenti possono leggere i file `machine.config` del framework .NET e `web.config` di root.
Non memorizzare informazioni sensibili in questi file se riguardano l'amministratore
- cifra le informazioni sensibili che dovrebbero essere lette solo dai worker process di IIS e non dagli altri utenti sulla macchina
- non assegnare l'accesso in Write all'identità che il web server usa per accedere a `applicationHost.config`.
Questa identità dovrebbe avere solo l'accesso in Read
- usa un'identità separata per pubblicare applicationHost.config.
Non usare quest'identità per configurare l'accesso alla configurazione condivisa sui web server
- usa una password forte quando esporti le chiavi di cifratura da usare per la condivisione della configurazione
- mantieni un accesso ristretto allo share contenente le configurazioni condivise e le chiavi di cifratura.
Se questo share viene compromesso, un attaccante sarà in grado di leggere e scrivere qualsiasi configurazione del tuo web server, redirigere traffico dal tuo sito a risorse malevole, e in alcuni casi ottenere il controllo di tutti i web server caricando codice arbitrario nel worker process di IIS
- valuta se proteggere questo share con regole firewall e policy IPsec per consentire solo ai web server membri di connettersi

#### Logging

Il logging è un asset importante della sicurezza di un'architettura di un'app, dato che può essere usato per individuare le vulnerabilità nelle app (gli utenti che provano a recuperare un file che non esiste) come anche gli attacchi tentati da un utente malevolo.
I log sono di solito generati adeguatamente dai software.
Non è comune trovare app che facciano il log adeguato delle loro azioni e, quando lo fanno, l'intenzione principale è produrre un output di debugging per consentire ai programmatori di analizzare un particolare errore.

In entrambi i casi (log di server e di app) bisognerebbe testare e analizzare diverse issue in base al contenuto dei log:

- i log contengono informazioni sensibili?
- i log sono memorizzati in un server dedicato?
- l'uso dei log può generare una condizione di DoS?
- come vengono ruotati? 
I log sono mantenuti per un adeguato periodo di tempo?
- come vengono verificati i log?
 Gli amministratori possono usare queste verifiche per individuare gli attacchi?
- come vengono conservati i backup dei log?
- i dati prima di essere inseriti nei log vengono validati?

#### Sensitive Information in Logs

Alcune app potrebbero, per esempio, usare le richieste GET per inoltrare i dati di un form che potrebbero essere memorizzati nei log del server.
Ciò significa che i log del server potrebbero contenere informazioni sensibili (come username e password, o dettagli di un account bancario).
Queste informazioni sensibili potrebbero essere usate da un attaccante se recupera i log, per esempio, tramite le interfacce amministrative o vulnerabilità note o mal configurazioni del web server (come la mal configurazione `server-status` nei server HTTP Apache).

I log degli eventi contengono spesso dati che sono utili all'attaccante (information leakage) o possono essere usati direttamente negli exploit:

- informazioni di debug
- stack trace
- username
- nomi di componenti del sistema
- indirizzi IP interni
- dati personali poco sensibili (es. indirizzi email, indirizzi postali, numeri di telefono associati a un individuo)
- dati aziendali

Inoltre, in alcune giurisdizioni, la memorizzazione di informazioni sensibili in file di log, come dati personali, 
potrebbe obbligare l'azienda ad applicare le leggi di protezione dei dati anche ai log come dovrebbero fare sui database di backend.
Non rispettando tali obblighi, anche senza saperlo, potrebbe comportare l'applicazione di penali all'azienda.

Una lista più ampia di informazioni sensibili è:

- codice sorgente dell'app
- valori di identificativi di sessione
- token di accesso
- dati personali sensibili e altre forme di PII
- password di autenticazione
- string di connessione al database
- chiavi di cifratura
- dati dei possessori di account bancari o carte di credito
- dati di un livello di sicurezza più alto rispetto a quelli che il sistema può memorizzare
- informazioni commerciali sensibili
- informazioni reperite ma non memorizzabili sotto la particolare giurisdizione
- informazioni per cui l'utente ha scelto di non farle memorizzare, o non ha dato il consenso, 
es. ha usato il do not track o il consenso è scaduto

#### Log Location

Di solito i server generano log locali contenenti le loro azioni e i loro errori, riempiendo il disco del sistema su cui il server è in esecuzione.
Comunque, se il server viene compromesso i suoi log possono essere cancellati dall'intruso per ripulire qualsiasi traccia del suo attacco e dei suoi metodi.
Se questo accadesse l'amministratore di sistema non avrebbe modo di sapere dell'attacco o della sua provenienza.
In realtà, molti tool di attacco includono un "log zapper" che è capace di ripulire i log che contengono certe informazioni (come gli indirizzi IP dell'attaccante).

Ne consegue che è più saggio memorizzare i log in una posizione diversa e non sullo stesso web server.
Ciò rende più facile aggregare i log da sorgenti diverse che provengono dalla stessa app (come quelli dalle web server farm) e rende più facile fare l'analisi dei log (che può essere CPU intensive) senza impattare sullo stesso server.

#### Log Storage

I log posono introdurre una condizione di DoS se non sono memorizzati in modo adeguato.
Qualsiasi attaccante con risorse sufficienti potrebbe produrre un sufficiente numero di richieste che potrebbero riempire lo spazio allocato per i file di log, se non sono stati preparati per evitare questa situazione.
Comunque, se il server non è configurato adeguatamente, i file di log sono memorizzati nella stessa partizione di disco usata dal software del SO o dall'app stesssa.
Ciò significa che se il disco si riempisse il SO o l'app potrebbero andare in crash a causa dell'impossibilità di scrivere su disco.

Di solito sistemi UNIX i log sono memorizzati in `/var` (anche se alcune installazioni di server potrebbero risiedere in `/opt` o `/usr/local`) 
ed è importante assicurarsi che le directory in cui i log sono memorizzati siano in una partizione diversa.
In alcuni casi, e per impedire che i log di sistema siano impattati, 
la directory dei log del software del server (come `/var/log/apache` per il web server Apache) dovrebbe essere memorizzata in una partizione separata.

Questo non per dire che i log dovrebbero crescere fino a ripempire il file system in cui si trovano.
La crescita dei log dovrebbe essere monitorata dato che potrebbe essere un indicatore di un attacco.

Il testing di questa condizione è così facile, e così pericolosa in ambienti di produzione, che 
basta inviare un numero sufficiente e sostenuto di richieste per vedere se queste richieste sono memorizzate nei log e se c'è possibilità di riempire la partizione.
In alcuni ambienti in cui anche i parametri QUERY_STRING sono memorizzati indipendentemente dal fatto che siano prodotti tramite richieste GET o POST, 
è possibile simulare grosse query che possono riempire i log più velocemente dato che, di solito, 
una richiesta singola comporterà al logging di una piccola quantità di dati, come la data e l'orario, l'indirizzo IP sorgente, l'URI, e il risultato del server.

#### Log Rotation

Molti server (ma poche app custom) ruotano i log per impedire che riempiano il file system in cui sono memorizzati.
L'assunzione è che le informazioni che contengono siano necessarie solo per un tempo limitato.

Questa feature dovrebbe essere testata per assicurarsi che:

- i log siano mantenuti per il tempo definito dalla security policy, non di più nè di meno
- i log siano compressi una volta ruotati 
(è conveniente, dato che più log verranno memorizzati nello stesso spazio disco)
- i permessi del file system di ruotare i file di log sono gli stessi (o più ristretti) di quelli del file di log stesso.
Per esempio, i web server hanno bisogno di scrivere sui log ma non hanno bisogno di scrivere sui log già ruotati, ciò significa che i permessi di questi file possono essere cambiati al momento della rotazione per impedire al processo del web server di modificarli

Alcuni server potrebbero ruotare i log quando raggiungono una certa dimensione.
Se succede, bisogna assicurarsi che un attaccante non possa forzare la rotazione al fine di coprire le sue tracce.

#### Log Access Control

I log non dovrebbero essere visibili agli utenti.
Anche gli amministratori web non dovrebbero vedere questi log dato che verrebbe rotto il vincolo di separation of duty.
Assicurarsi che 
qualsiasi schema di access control che sia usato per proteggere l'accesso ai log o 
qualsiasi app che permette di vedere o cercare i log 
non sia collegata a schema di access control per altri user role applicativi.
I log non dovrebbero essere visibili nemmeno a utenti non autenticati.

#### Log Review

La verifica dei log può essere usata oltre che per l'estrazione di statistiche di utilizzo dei file in un sever web (che di solito è quello su cui le app basate sui log si focalizzano) 
anche per determinare se un attacco è stato realizzato contro il web server.

Per analizzare gli attacchi al web server, devono essere analizzati i file di log del server.
La verifica dovrebbe concentrarsi su:

- messaggi di errore 40x.
Un elevato numero di questi messaggi dalla stessa sorgente potrebbero indicare che è stato usato uno scanner CGI contro il web server
- messaggi di errore 50x.
Possono essere un indicatore di un attacco che abusa di parti dell'app che non sono in grado di gestire le richieste.
Per esempio, le prime fasi di un attacco di SQL injection produrranno questi messaggi di errore 
quando la query SQL non è costruita adeguatamente e la sua esecuzione fallisce sul database di backend

Le statistiche e le analisi dei log non dovrebbero essere generate o memorizzate sullo stesso server che produce i log.
Altrimenti, un  attaccante potrebbe, tramite una vulnerabilità del web server o una mal configurazione, 
ottenere l'accesso e recuperare informazioni simili a quelle ottenibili dai log stessi.