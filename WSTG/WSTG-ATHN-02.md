# WSTG-ATHN-02

# Testing for Default Credentials

## Summary

Al giorno d'oggi le web app usano software open source o commerciale che può essere installato sui server con una configurazione minima o personalizzabile dall'amministratore del server.
Inoltre, molte hardware appliance (es. router e database server) offrono una configurazione web-based o un interfaccia amministrativa.

Spesso queste app, una volta installate, non sono configurate adeguatamente e le credenziali di default fornite per l'autenticazione e la configurazione iniziale non vengono cambiate.
Queste credenziali di default sono note ai penetration tester, e sfortunatamente, anche agli attaccanti, che possono usarle per accedere.

Inoltre, in molte situazioni, quando viene creato un nuovo account in un'app, viene gerata una password di default (con delle caratteristiche standard).
Se questa password è predicibile e l'utente non la cambia dopo il primo accesso, l'attaccante può ottenere un accesso non autorizzato all'app.

L'origine di questo problema può essere identificato come:

- personale IT senza esperienza, non cosciente dell'importanza del cambio delle password di default sui componenti installati, o che lascia la password di default per "ease of maintenance"
- programmatori che lasciano back door per accedere facilmente e testare la loro app ma dimenticano di rimuoverla
- app con account di default built-in non rimuovibili con username e password predefiniti
- app che non forzano l'utente a cambiare le credenziali di default dopo il primo login

## How to Test

### Black-Box Testing

### Testing for Default Credentials of Common Applications

Nel black-box testing, il tester non sa nulla sull'app e sull'infrastruttura sottostante.
In realtà questo non è sempre vero, infatti conosce alcune informazioni sull'app.
Supponiamo che tu abbia identificato, tramite le tecniche descritte in questa guida nel capitolo Information Gathering, almeno una o più app comuni che potrebbero contenere interfacce amministrative.

Quando hai identificato un'interfaccia dell'app, per esempio un'interfaccia web di un router Cisco o un portale amministrativo WebLogic, controlla se gli username e le password noti per questi device siano utilizzabili.
Per fare ciò puoi consultare la documentazione del produttore o, più semplicemente, puoi trovare le credenziali note usando un motore di ricerca o usando uno dei siti o dei tool elencati di seguito.

Quando analizziamo app che non hanno una lista di account utente di default (per esempio perchè l'app non è diffusa) possiamo provare a indovinare le credenziali di default valide.
Nota che l'app potrebbe avere una policy di lockout dell'account, e più tentativi per indovinare la password contro un username noto potrebbero bloccarlo.
Se è possibile bloccare l'account amministratore, per l'amministratore di sistema potrebbe essere problematico reimpostarlo.

Molte app generano messaggi di errore verbosi che informano se gli username usati sono validi.
Queste informazioni saranno utili durante il testing degli account utente di default.
Queste funzionalità possono essere trovate, per esempio, nella pagina di login, nella paina di reset o recupero password, e nella pagina di registrazione.
Una volta che hai trovato un username di default puoi iniziare a indovinare la password per questo account.

Maggiori informazioni su questa procedura possono essere trovate nella sezione [Testing for Account Enumeration and Guessable User Account](./WSTG-IDNT-04.md) e nella sezione [Testing for Weak Password Policy](./WSTG-ATHN-07.md).

Dato che questi tipi di credenziali di default sono legati agli account amministrativi puoi procedere in questo modo:

- prova i seguenti username: 
"admin",
"administrator",
"root",
"system",
"guest",
"operator",
"super".
Sono spesso usati dagli amministratori di sistema.
Inoltre potresti provare:
"qa",
"test",
"test1",
"testing"
e nomi simili.
Prova qualsiasi combinazione dei precedenti sia nei campi dell'username che della password.
Se l'app è vulnerabile all'username enumeration, e sei in grado di identificare uno dei precedenti username, prova le password nello stesso modo.
Inoltre prova una password vuota o una delle seguenti
"password",
"pass123",
"password123",
"admin",
"guest"
con gli account precedenti o qualsiasi altro account individuato.
Puoi provare altre permutazioni delle precedenti.
Se queste password non vanno bene, varrebbe la pena usare una lista di username e password conosciute e provare più richieste nei confronti dell'app.
Si può creare uno script per risparmiare tempo
- gli utenti amministrativi dell'app di solito hanno il nome che coincide con il nome dell'app o dell'azienda.
Ciò significa che se stai testando l'app "Obscurity", prova a usare obscurity/obscurity o altre simili combinazioni come username e password
- quando esegui un test per un cliente, prova a usare i nomi dei contatti che hai ricevuto come username con qualsiasi password comune.
Gli indirizzi email del cliente rivelano la naming convention degli account utente:
se l'impiegato John Doe ha l'indirizzo email jdoe@example.com, puoi provare a cercare i nomi degli amministratori del sistema sui social media e indovinare la loro username applicando la stessa naming convention con il loro nome
- prova a usare gli username precedenti con la password vuota
- analizza la pagina e il JavaScript tramite un proxy o ispezionando il sorgente HTML.
Cerca qualsiasi riferimento a utenti e password nel sorgente.
Per esempio `if username='admin' then starturl=/admin.asp else startrul=/index.asp` (login avvenuto con successo vs fallito).
Inoltre, se hai un account valido, accedi e cerca nelle richieste e nelle risposte di un login avvenuto con successo vs fallito dei campi nascosti addizionali, richieste GET interessanti (es. login=yes), ecc.
- cerca nomi di account e password scritti nei commenti del codice sorgente.
Cerca anche nelle directory di backup il codice sorgente (o backup del codice sorgente) che potrebbero contentere commenti e codice interessanti

#### Testing for Default Password of New Accounts

Può anche succedere che quando viene creato un nuovo account gli venga assegnato una password di default.
Questa password potrebbe avere delle caratteristiche standard che la rendono predicibile.
Se l'utente non la cambia al suo primo accesso (ciò succede se l'utente non è costretto a cambiarla) o se l'utente non ha mai eseguito l'accesso all'app, allora l'attaccante può ottenere un accesso non autorizzato all'app.

Il consiglio precedente sulla possibile policy di lockout e sui messaggi di errore verbosi sono anche applicabili qui per il testing sulle password di default.

I seguenti passi possono essere applicati per le credenziali di default:

- l'ispezione della pagina di registrazione utente potrebbe aiutare a determinare il formato e la lunghezza minima e massima delle username e delle password dell'app.
Se la pagina di registrazione utente non esiste, determina se l'azienda usa una naming convention standard per i nomi degli utenti come i loro indirizzi email o il nome prima della `@` nell'email
- prova a capire come sono generati gli username.
Per esempio, un utente può scegliere il suo username o il sistema lo genera in base ad alcune informazioni personali dell'utente o usando una sequenza predicibile?
Se l'app genera gli username in modo predicibile, come `user7811`, prova a fare fuzzing su tutti i possibili account.
Se riesci a individuare una risposta diversa dall'app quando si usa un username valido e una password errata, puoi provare a fare un attacco di bruteforce sull'username valido (o provare le password comuni identificate precedentemente)
- prova a capire se il sistema genera password predicibili.
Per fare ciò, crea velocemente molti account in modo da confrontare e capire se le password sono predicibili.
Se lo sono, prova a correlarle con gli username, o qualsiasi account enumerato, e usale come base per l'attacco di bruteforce
- se riesci a identificare la corretta naming convention per gli username, prova a fare il brute force della password con sequenze predicibili come l'esempio delle date di nascita
- prova a usare tutti gli username precedenti con una password vuota e usare l'username come password

### Gray-Box Testing

I seguenti passi rispettano un approccio interamente grey-box.
Se hai a disposizione solo alcune di queste informazioni, fai riferimento alla sezione black-box per colmare le lacune.

- parla con il personale IT per capire quali password usano per l'accesso amministrativo e come viene gestita l'amministrazione dell'app
- chiedi al personale IT se le password di default sono cambiate e se gli account di default sono disabilitati
- cerca nel database degli utenti credenziali di default come descritto nella sezione black-box testing.
Cerca anche password vuote
- esamina il codice per username e password hardcoded
- controlla se i file di configurazione contengono username e password
- esamina la password policy e, se l'app genera le sue password per i nuovi utenti, verifica la policy in uso per questa procedura

## Tools

- Burp Intruder
- THC Hydra
- Nikto 2