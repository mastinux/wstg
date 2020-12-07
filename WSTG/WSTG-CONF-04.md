# WSTG-CONF-04

# Review Old Backup and Unreferenced Files for Sensitive Information

## Summary

Anche se la maggior parte dei file in un web server sono sono gestiti direttamente dal server stesso, 
è possibile trovare file non referenziati o dimenticati che possono essere usati per ottenere informazioni importanti sull'infrastruttura o sulle credenziali.

Gli scenari più comuni includono la presenza di versioni vecchie di file modificati, file che sono caricati dal linguaggio e possono essere scaricati come sorgente, o anche backup automatici o manuali nella forma di archivi compressi.
I file di backup possono anche essere generati automaticamente dal file system sottostante l'app, una feature di solito definita "snaptshot".

Tutti questi file potrebbero fornire al tester l'accesso a progetti interni, a back door, a interfacce amministrative, o anche a credenziali per connettersi all'interfaccia amministrativa o al database server.

Un'importante sorgente di vulnerabilità risiede nei file che non hanno a che fare con l'app, ma sono creati come una conseguenza della modifica dei file dell'app, o della creazione di copie di backup, o della mancata rimozione di file vecchio o non referenziati nell'albero del web server.
L'esecuzione di modifiche o altre azioni amministrative su web server di produzione potrebbe lasciare inavvertitamente copie di backup, generate automaticamente dall'editor durante la modifica dei file, o dall'amministratore che crea uno zip dei file per fare un backup.

È facile dimenticare questi file e potrebbero costituire un seria minaccia per l'app.
Ciò accade perchè le copie di backup potrebbero essere generate con estensioni di file diverse da quelle dei file originali.
Un archivio `.tar`, `.zip` o `.gz` che generiamo (e che dimentichiamo ...) ha un'estensione diversa, e lo stesso accade con le copie automatiche create dai vari editor (per esempio, emacs genera una copia di backup chiamata `file~` quando si modifica `file`).
La copia manuale potrebbe produrre lo stesso effetto (pensa alla copia di `file` in `file.old`).
Il file system sottostante l'app potrebbe fare degli snapshot dell'app in diversi momenti senza avvertirci, che potrebbero essere accessibili via web, comportando una minaccia simile a quella del file di backup.

Risulta che, queste attività generano file che non sono necessari all'app e potrebbero essere gestiti diversamente rispetto ai file originali dal web server.
Per esempio, se facciamo una copia di `login.asp` col nome `login.asp.old`, consentiamo agli utenti di scaricare il contenuto del codice sorgente di `login.asp`.
Questo perchè `login.asp.old` verrà di solito restituita come testo, piuttosto che essere eseguita a causa della sua estensione.
In altre parole, l'accesso a `login.asp` comporta la sua esecuzione lato server, mentre l'accesso a `login.asp.old` comporta la restituzione del suo contenuto in formato testuale.
Ciò potrebbe costituire un rischio di sicurezza, dato che potrebbero essere rilevate informazioni sensibili.

In generale, l'esposizione del codice sorgente del server è una cattiva idea.
Non solo stai esponendo la business logic senza motivo, ma potresti rilevare informazioni relative all'app che potrebbero aiutare un attaccante (path, strutture dati, etc.).
Ricordiamo molti script hanno username e password embedded in clear text (che è una pratica negligente e pericolosa).

Gli altri casi di file non referenziati sono dovuti a scelte di progettazione o di configurazione che consentono che 
diversi tipi di file relativi all'app come file dati, file di configurazione, file di log, siano memorizzati nelle directory del file system e possano essere restituiti dal web server.
Non ha senso che questi file si trovino nel file system, che può essere acceduto tramite web, 
dato che dovrebbero essere acceduti solo a livello di app, dall'app stessa (e non da un utente che sta accedendo al sito).

## Threats

I file vecchi, di backup o non referenziati comportano varie minacce alla sicurezza di una web app:

- i file non referenziati potrebbero esporre informazioni sensibili che possono facilitare un attacco focalizzato contro l'app;
per esempio questi includono 
i file contenenti credenziali di database, 
i file di configurazione contenenti riferimenti a altri contenuti nascosti,
i path assoluti a file, etc.
- le pagine non referenziate potrebbero contenere funzionalità potenti che possono essere usate per attaccare l'app;
per esempio una pagina amministrativa che non è referenziata dal contenuto pubblicato ma che può essere acceduta da qualsiasi utente sappia come trovarla
- i file vecchi e di backup potrebbero contentere vulnerabilità che sono state risolte nelle versioni più recenti;
per esempio `viewdoc.old.jsp` potrebbe contenere una vulnerabilità di directory traversal che è stata risolta in `viewdoc.jsp` 
ma può essere sfruttata da chiunque trovi la vecchia versione 
- i file di backup potrebbero rivelare il codice sorgente di pagine progettate per essere eseguite sul server;
per esempio richiedendo `viewdoc.bak` potrebbe restituire il codice sorgente di `viewdoc.jsp`, che può essere controllato per trovare vulnerabilità che potrebbero essere difficili da trovare tramite l'invio di richieste alla cieca.
Anche se questa minaccia riguarda i linguaggi di scripting, come Perl, PHP, ASP, script shell, JSP, etc. non è limitato ad essi, come mostrato in uno degli esempi che seguono.
- gli archivi di backup potrebbero contenere tutti i file all'interno (o anche all'esterno) della webroot.
Ciò consente all'attaccante di enumerare velocemente tutta l'app, incluse le pagine non referenziate, il codice sorgente, etc.
Per esempio, se ti dimentichi del file `myservlets.jar.old` contenente le classi di implementazione dei tuoi servlet, 
stai esponendo molte informazioni sensibili che possono essere sottoposte a decompilazione e a reverse engineering
- in alcuni casi la copia o la modifica di file non modifica la sua estensione, ma modifica il suo nome.
Ciò accade per esempio nell'ambiente Windows, le operazioni di copia di un file generano un file con il prefisso "Copy of" o versioni localized del suo nome.
Dato che l'estensione del file non cambia, non è il caso in cui un file eseguibile viene restituito come testo dal server, e non un caso di esposizione di codice sorgente.
Comunque, anche questi file sono pericolosi perchè esiste la possibilità che contengano logiche obsolete o incorrette, che quando vengono invocate, potrebbero scatenare errori applicativi, che potrebbero esporre informazioni di valore per l'attaccante, se i messaggi di diagnostica sono abilitati
- i file di log potrebbero contenere informazioni sensibili sulle attività degli utenti dell'app, per esempio i dati sensibili passati nei parametri dell'URL, nei session ID, nelle URL visitate (che potrebbero rivelare contenuti non referenziati), etc.
Gli altri file di log (es. log ftp) potrebbero contenere informazioni sensibili sulla manutenzione dell'app da parte degli amministratori
- gli snapshot del file system potrebbero contenere copie del codice che contengono vulnerabilità che sono state risolte in versioni più recenti.
Per esempio `/.snapshot/montly.1/view.php` potrebbe contentere una vulnerabilità di directory traversal che è stata risolta in `/view.php` ma che è ancora sfruttabile da chiunque trovi la vecchia versione

## How to Test

### Black-Box Testing

Il testing di file non referenziati usa sia teniche automatiche che manuali, e di solito adotta una combinazione delle seguenti:

#### Inherence from the Naming Scheme Used for Published Content

Enumera tutte le pagine e le funzionalità dell'app.
Puoi farlo manualmente usando un browser, o usando un tool di spidering.
Molte app usano un naming scheme riconoscibile, e organizzano le risorse in pagine e directory usando parole che descrivono la loro funzione.
In base al naming scheme usato per pubblicare il contenuto, è spesso possibile risalire al nome e alla posizione di pagine non referenziate.
Per esempio, se viene trovata una pagina `viewuser.asp`, allora cerca anche `edituser.asp`, `adduser.asp` e `deleteuser.asp`.
Se trovi la directory `/app/user`, cerca anche `/app/admin` e `/app/manager`.

#### Other Clues in Published Content

Molte web app lasciano indizi sul contenuto pubblicato che può portare alla rivelazione di pagine e funzionalità nascote.
Questi indizi di solito appaiono nel codice sorgente HTML o nei file JavaScript.
Questo codice sorgente deve essere controllato manualmente prima di essere pubblicato per identificare eventuali indizi su pagine o funzionalità.
Per esempio:

I commenti dei programmatori nel codice sorgente potrebbero far riferimento a contenuti nascosti:

```xml
<!-- <A HREF="uploadfile.jsp">Upload a document to the server</A> -->
<!-- Link removed while bugs in uploadfile.jsp are fixed -->
```

Il JavaScript potrebbe contenere link a pagine che sono solo renderizzate dalla GUI solo sotto particolari circostanze:

```javascript
var adminUser=false;
if (adminUser) menu.add (new menuItem ("Maintain users", "/admin/useradmin.jsp"));
```

Le pagine HTML potrebbero contenere form che sono stati nascosti disabilitando l'elemento submit:

```html
<FORM action="forgotPassword.jsp" method="post">
	<INPUT type="hidden" name="userID" value="123">
	<!-- <INPUT type="submit" value="Forgot Password"> -->
</FORM>
```

Un'altra sorgente di indizi sulle directory non referenizate è il file `robots.txt` usato per istruire i web robot:

```
User-agent: *
Disallow: /Admin
Disallow: /uploads
Disallow: /backup
Disallow: /~jbloggs
Disallow: /include
```

#### Blind Guessing

Nella sua forma più semplice, consiste nell'usare una lista di nomi di file comuni tramite più richieste nel tentativo di individuare i file e le directory che esistono sul server.
Il seguente script usa netcat per eseguire un guessing attack a partire da una wordlist da stdin:

```sh
#!/bin/bash

server=example.org
port=80

while read url
do
	echo -ne "$url\t"
	echo -e "GET /$url HTTP/1.0\nHost: $server\n" | netcat $server $port | head -1
done | tee outputfile
```

In base al server, GET potrebbe essere sostituito con HEAD per risultati più veloci.
Si potrebbe eseguire un grep sul file di output alla ricerca di codici di risposta "interessanti".
Il codice di risposta 200 (OK) di solito indica che è stata trovata una risorsa valida (assunto che il server non restituisca una pagina custom di "not found" usando il codice 200).
Ma cerca anche i codici 301 (Moved), 302 (Found), 401 (Unauthorized), 403 (Forbidden) e 500 (Internal error), che potrebbero indicare risorse o directory che meritano un'ulteriore investigazione.

Il guessing attack mostrato prima dovrebbe essere eseguito contro la webroot, e anche contro tutte le directory che sono state identificate tramite altre tecniche di enumerazione.
Altri guessing attack più avanzati/efficaci possono essere eseguiti come segue:

- identifica le estensioni dei file in are note dell'app (es. jsp, aspx, html) e 
usa una wordlist base concatenando ognuna di queste estensioni (o usa una lista di estensioni più lunga se le risorse te lo permettono)
- per ogni file identificato tramite le altre tecniche di enumerazione, crea una wordlist comune derivata da quei nomi di file.
Ottieni una lista di estensioni di file comuni (inclusi `~`, `bak`, `txt`, `src`, `dev`, `old`, `inc`, `orig`, `copy`, `tmp`, `swp`, etc.) e 
inserisci ogni estensione prima, dopo e al posto di ogni estensione del file esistente

Nota:
le operazioni di copia in Windows generano file col prefisso "Copy of" o versioni localized del suo nome, quindi non cambiano l'estensione del file.
Anche se i file "Copy of" di solito non rivelano codice sorgente quando vengono acceduti, potrebbero fornire informazioni di valore quando generano un errore se vengono invocate.

#### Information Obtained Through Server Vulnerabilities and Misconfiguration

Il modo più ovvio tramite cui un server malconfigurato può rivelare pagine non referenziate è attraverso il directory listing.
Invia richieste per ogni directory enumerata per identificare quelle che forniscono il directory listing.

Sono state trovate numerose vulnerabilità in web server che consentivano all'attaccante di enumerare il contenuto non referenziato, per esempio:

- vulnerabilità Apache di directory listing ?M=D
- diverse vulnerabilità di disclosure del sorgente di script IIS
- vulnerabilità di directory listing WebDAV IIS

#### Use of Publicly Available Information

Le pagine e le funzionalità nelle web app affacciate su Internet non referenziate all'interno dell'app stessa potrebbero essere referenziate da altre sorgenti pubbliche.
Ci sono diverse risorse a riguardo:

- le pagine che erano referenziate possono ancora comparire negli archivi dei motori di ricerca.
Per esempio, `1998results.asp` potrebbe non essere più referenziato sul sito di un'azienda, ma potrebbe rimanere sul server e nei database dei motori di ricerca.
Questo vecchio script potrebbe contenere vulnerabilità che potrebbero essere usate per compromettere l'intero sito.
L'operator `site:` può essere usato per eseguire una query solo rispetto al dominio scelto, come ad esempio `site:www.example.com`.
L'uso dei motori di ricerca in questo modo può fornire un insieme di tecniche che possono essere utili e che sono descritte nella sezione `Google Hacking` di questa guida.
Dai un'occhiata per affinre le tue skill di ricerca su Google.
È difficile che i file di backup siano referenziati da altri file e che siano quindi stati indicizzati da google, ma se risiedono in una directory navigabile è possibile che il motore di ricerca ne conosca l'esistenza
- inoltre, Google e Yahoo mantengono versioni cached delle pagine trovate dai loro robot.
Anche se `1998results.asp` è stato rimosso dal server target, una versione del suo output potrebbe essere ancora memorizzata in questi motori di ricerca.
La versione cached potrebbe contenere riferimenti a, o indizi su, altri contenuti nascosti che sono ancora presenti sul server
- il contenuto che non è referenziato dall'app target potrebbe essere referenziato da siti di terze parti.
Per esempio, un'app che processa i pagamenti online per conto di terze parti potrebbe contenere funzionalità che possono (normalmente) essere trovate solo seguendo i link nel sito web dei suoi clienti

#### File Name Filter Bypass

Dato che i filtri basati su blacklist sono basati su regular expression, 
si potrebbe sfruttare la feature di filename expansion del SO in un modo che il programmatore non si aspettava.
Il tester può a volte sfruttare i modi diversi in cui i filename sono parsed dall'app, dal web server, o dal SO sottostante e le convensioni sui file name.

Esempio: secondo la filename expansion di Windows 8.3 `c:\\program files` diventa `c:\\PROGRA\~1`

- rimuove i caratteri non compatibili
- converte gli spazi in underscore
- prende i primi sei caratteri del nome
- aggiunge `~<numero>` che serve a distinguere file che hanno i primi sei caratteri uguali
- questa convensione cambia dopo le prime tre collisioni di cname
- tronca l'estensione dei file ai tre caratteri
- rende tutti i caratteri maiuscoli

### Gray-Box Testing

L'esecuzione di gray box testing contro file vecchi e di backup neccessita l'analisi dei file contenuti nelle directory appartenenti all'insieme delle web directory servite dal web server dell'infrastruttura della web app.
Teoricamente l'analisi dovrebbe essere eseguita a mano per essere accurata.
Comunque, dato che in molti casi le copie dei file o i backup dei file tendono a essere creati usando la stessa naming convention, la ricerca può essere facilmente scripted.
Per esempio, gli editor lasciano copie di backup rinominandole con estensioni riconoscibili e si tende a lasciare i file con estensioni predicibili o estensione `.old`.
Una buona strategia è 
predisporre un job periodico in background che cerca file con estensioni che sono di backup o copie di file, 
ed eseguire un controllo manuale nel lungo periodo.

### Tools

I tool di assessment di vulnerabilità tendono a includere i controlli per individuare web directory che hanno nomi standard (come "admin", "test", "backup", etc.), e a riportare qualsiasi web directory che consente il listing.
Se non ottieni nessun directory listing, dovresti provare con le estensioni dei backup. 

- Nessus
- Nikto2

Tool di web spidering

- wget
- Wget for Windows
- Sam Spade
- Spike proxy
- Xenu
- curl

Alcuni di questi sono inclusi nelle distribuzioni standard di Linux.
I tool di sviluppo web includono funzionalità per individuare file non referenziati.

## Remediation

Per garantire una strategia di protezione efficace, il testing dovrebbe essere composto da una security policy che vieta esplicitamente pratiche pericolose, come:

- la modifica di file presenti sul file system del web server o dell'app server.
Questa è una cattiva abitudine, dato che vengono involontariamente generati file di backup da parte dell'editor.
È sorprendente quanto spesso questo venga fatto, anche in grandi organizzazioni.
Se devi assolutamente modificare i file su un sistema di produzione, 
assicurati che non venga lasciato nulla che non sia esplicitamente necessario, 
e considera che lo stai facendo a tuo rischio
- controlla attentamente qualsiasi altra attività eseguita sul file system che è esposta al web server, come attività di amministrative saltuarie.
Per esempio, se devi fare occasionalmente uno snapshot di un paio di directory (che non dovresti fare in un sistema di produzione), potresti essere tentato nel farne uno zip.
Fai attenzione a non lasciare questi file nel file system
- le policy di configuration management appropriate dovrebbero aiutare a non lasciare file obsoleti e non referenziati
- le app dovrebbero essere progettate per non creare (o basarsi su) file memorizzati sotto le directory servite dal web server.
File di dati, file di log, file di configurazione, etc. dovrebbero essere memorizzati in directory non accessibili dal web server, 
per contrastare la possibilità di information disclosure (per non citare la modifica dei dati se i permessi sulla web directory consentono la scrittura)
- gli snapshot dei file system non dovrebbero essere accessibili tramite web se il document root è nel file system che usa questa tecnologia.
Configura il tuo web server per impedire l'accesso a queste directory, per esempio in Apache dovrebbe essere usata una direttiva `location` come la seguente:

```
<Location ~ ".snapshot">
	Order deny,allow
	Deny from all
</Location>
```