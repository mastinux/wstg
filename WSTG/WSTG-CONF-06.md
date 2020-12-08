# WSTG-CONF-06

# Test HTTP Methods

## Summary

HTTP offre una serie di metodi che possono essere usati per eseguire azioni sul web server.
Molti di questi metodi sono progettati per aiutare gli sviluppatori a pubblicare e a testare le app HTTP.
Questi metodi HTTP possono essere usati per scopi malevoli se il web server è malconfigurato.
Più avanti, viene esaminato il Cross Site Tracing (XST), una forma di XSS che sfrutta il metodo HTTP TRACE.
Mentre GET e POST sono i metodi più comuni usati per accedere alle informazioni fornite dal web server, HTTP consente l'utilizzo di altri metodi.
L'RFC 2616 (che descrive la versione 1.1 di HTTP) definisce i seguenti otto metodi:

- HEAD
- GET
- POST
- PUT
- DELETE
- TRACE
- OPTIONS
- CONNECT

Alcuni di questi metodi possono presentare un rischio di sicurezza per la web app, 
dato che consentono all'attaccante di modificare i file memorizzati sul web server e, in alcuni scenari, di rubare le credenziali di utenti legittimi.
Più specificatamente, i metodi che dovrebbero essere disabilitati sono:

- PUT:
questo metodo consente al client di caricare nuovi file sul web server.
Un attaccante può sfruttarlo per caricare file malevoli (es. un file asp che esegue dei comandi invocando cmd.exe), o semplicemente usando il server della vittima come repository di file
- DELETE:
questo metodo consente al client di eliminare un file dal web server.
Un attaccante può sfruttarlo in modo semplice e diretto per fare il defacement del sito o per realizzare un attacco DoS
- CONNECT:
questo metodo può consentire al client di usare il web server come un proxy
- TRACE:
questo meotodo restituisce al client quello che è stato inviato al server, e viene usato a scopi di debug.
Questo metodo, originariamente considerato inoffensivo, può essere usato per realizzare un attacco chiamato Cross Site Tracing

Se un'app ha bisogno di uno o più di questi metodi, come nel caso dei REST Web Services (che potrebbero richiedere PUT e DELETE), 
è importante controllare che il loro utilizzo sia limitato agli utenti fidati e in codizioni sicure.

### Arbitrary HTTP Methods

Arshan Dabisriaghi ha scoperto che molti web app framework consentono ai metodi standard o a dei metodi custom di raggirare i filtri di access control:

- molti framework e linguaggi trattano la richiesta HEAD come una GET, anche se senza body nella risposta.
Se è stato imposto un vincolo sui metodi GET in modo che solo gli "authenticatedUsers" possano accedere con richieste GET a particolari risorse o servlet, potrebbe essere raggirato usando HEAD.
Ciò consente la sottomissione di richieste GET privilegiate in modo non autorizzato
- alcuni framework accettano metodi HTTP arbitrari come JEFF o CATS senza alcuna limitazione.
Vengono trattati come una richiesta GET, e potrebbero non sottostare ai filtri di access control, consentendo la sottomissione di richieste GET privilegiate non autorizzate

In molti casi, il codice che controlla i metodi GET e POST dovrebbe essere sicuro.

## How to Test

### Discover the Supported Methods

Per eseguire questo test, il tester deve capire quali metodi HTTP sono supportati dal web server che sta esaminando.
Il metodo HTTP OPTIONS consente al tester di individuare in modo diretto ed efficace i metodi supportati.
L'RFC 2616 afferma: 
"The OPTIONS method represents a request for information about the communication options available on the request/response chain identified by the Request-URI".

La modalità di test è estremamente semplice e basta usare netcat (o telnet):

```sh
$ nc www.victim.com 80
OPTIONS / HTTP/1.1
Host: www.victim.com


HTTP/1.1 200 OK
Server: Microsoft-IIS/5.0
Date: Tue, 31 Oct 2006 08:00:29 GMT
Connection: close
Allow: GET, HEAD, POST, TRACE, OPTIONS
Content-Length: 0
```

Come possiamo vedere nell'esempio, OPTIONS fornisce una lista di metodi che sono supportati dal web server, e in questo caso possiamo vedere che il metodo TRACE è abilitato.
Il pericolo derivante da questo metodo è illustrato nella sezione successiva.

Lo stesso test può essere eseguito con nmap e con lo script NSE http-methods:

```sh
$ nmap -p 443 --script http-methods localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2015-11-04 11:52 Romance Standard Time

Nmap scan report for localhost (127.0.0.1)
Host is up (0.0094s latency).
PORT
STATE SERVICE
443/tcp open https
| http-methods: OPTIONS TRACE GET HEAD POST
| Potentially risky methods: TRACE
|_See http://nmap.org/nsedoc/scripts/http-methods.html

Nmap done: 1 IP address (1 host up) scanned in 20.48 seconds
```

### Test XST Potential

Nota: per capire la logica e l'obiettivo di quest'attacco dovresti prendere familiarità con l'attacco di XSS.

Il metodo TRACE, anche se apparentemente innocuo, può essere sfruttato con successo in alcuni scenari per rubare le credenziali di utenti legittimi.
Questa tecnica di attacco è stata scoperta da Jeremiah Grossman nel 2003, nel tentativo di raggirare il tag HTTPOnly che Microsoft aveva introdotto in Internet Explore 6 SP1 per impedire che i cookie fossero acceduti da JavaScript.
Difatti, uno dei pattern di attacco più ricorrenti nei XSS è l'accesso all'oggetto document.cookie e l'invio del suo contenuto a un server controllato dall'attaccante in modo da dirottare la sessione della vittima.
Contrassegnando un cookie con httpOnly si impedisce a JavaScript di potervi accedere, e quindi di poterlo inviare a terze parti.
Comunque, il metodo TRACE può essere usato per raggirare questa protezione e accedere al cookie anche in questo scenario.

Come citato prima, TRACE restituisce qualsiasi stringa che è stata inviata al web server.
Per verificare la sua presenza (o per verificare i risultati della richiesta OPTIONS), il tester può procedere come mostrato nel seguente esempio:

```sh
$ nc www.victim.com 80
TRACE / HTTP/1.1
Host: www.victim.com


HTTP/1.1 200 OK
Server: Microsoft-IIS/5.0
Date: Tue, 31 Oct 2006 08:01:48 GMT
Connection: close
Content-Type: message/http
Content-Length: 39

TRACE / HTTP/1.1
Host: www.victim.com
```

Il response body è esattamente una copia della richiesta originale, ciò significa che il server consente l'uso di questo metodo.
Ora, dove si nasconde il pericolo?
Se il tester fa eseguire una richiesta TRACE al browser verso il web server, e il browser ha un cookie per quel dominio, il cookie verrà incluso negli header della richiesta, e sarà restituito nella risposta del server.
A questo punto, il cookie sarà accedibile da JavaScript e sarà possibile inviarlo a terze parti anche se il cookie è contrassegnato con httpOnly.

Ci sono diversi modi per far inviare una richiesta TRACE dal browser, come un controllo ActiveX XMLHTTP in Internet Explorer e XMLDOM in Mozilla e Netscape.
Comunque, per ragioni di sicurezza il browser può avviare una connessione solo verso il dominio su cui risiede lo script malevolo.
È un fattore di mitigazione, dato che l'attaccante deve combiare il metodo TRACE con un'altra vulnerabilità per realizzare l'attacco.

Un attaccante ha due modi per lanciare con successo un attacco di XST:

- sfruttare un'altra vulnerabilità lato server:
l'attaccante inietta lo script JavaScript malevolo che contiene la richiesta TRACE nell'app vulnerabile, come un normale attacco di XSS
- sfruttare una vulnerabilità lato client:
l'attaccante crea un sito web malevolo che contiene lo script JavaScript malevolo 
e sfrutta una qualche vulnerabilità cross-domain del browser della vittima, 
per consentire al codice JavaScript di instaurare una connessione col sito che supporta il metodo TRACE e che genera il cookie che l'attaccante sta cercando di rubare

### Testing for Arbitrary HTTP Methods

Trova una pagina da visitare che ha dei vincoli di sicurezza che normalmente generano un 302 redirect verso una pagina di login o che forza a fare direttamente il login.
L'URL nell'esempio funziona in questo modo, come in molte web app.
Comunque, se ottieni la risposta 200 che non è la pagina di login, potrebbe essere possibile raggirare l'autenticazione e quindi l'autorizzazione.

```sh
$ nc www.example.com 80
JEFF / HTTP/1.1
Host: www.example.com


HTTP/1.1 200 OK
Date: Mon, 18 Aug 2008 22:38:40 GMT
Server: Apache
Set-Cookie: PHPSESSID=K53QW...
```

Se il framework o il firewall o l'app non supporta il metodo JEFF, dovrebbe mostrare una pagina di errore (o preferibilmente una pagina di 405 Not Allowed o 501 Not Implemented).
Se serve la richiesta, è vulnerabile a questo problema.

Se il tester verifica che il sistema è vulnerabile a questo problema, dovrebbero realizzare un attacco simile al CSRF per sfruttare il problema:

- FOOBAR /admin/createUser.php?member=myAdmin
- JEFF /admin/changePw.php?member=myAdmin&passwd=foo123&confirm=foo123
- CATS /admin/groupEdit.php?group=Admin&member=myAdmin&action=add

Con un po' di fortuna, usando i tre comandi precedenti - modificati in modo da adattarsi all'app e ai requisiti del test - dovrebbe essere creato un nuovo utente, con una password e di livello amministratore.

### Testing for HEAD Access Control Bypass

Trova una pagina da visitare che ha dei vincoli di sicurezza che normalmente generano un 302 redirect verso una pagina di login o che forza a fare direttamente il login.
L'URL nell'esempio funziona in questo modo, come in molte web app.
Comunque, se ottieni la risposta 200 che non è la pagina di login, potrebbe essere possibile raggirare l'autenticazione e quindi l'autorizzazione.

```sh
$ nc www.example.com 80
HEAD /admin HTTP/1.1
Host: www.example.com


HTTP/1.1 200 OK
Date: Mon, 18 Aug 2008 22:44:11 GMT
Server: Apache
Set-Cookie: PHPSESSID=pKi...; path=/; HttpOnly
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Set-Cookie: adminOnlyCookie1=...; expires=Tue, 18-Aug-2009 22:44:31 GMT; domain=www.example.com
Set-Cookie: adminOnlyCookie2=...; expires=Mon, 18-Aug-2008 22:54:31 GMT; domain=www.example.com
Set-Cookie: adminOnlyCookie3=...; expires=Sun, 19-Aug-2007 22:44:30 GMT; domain=www.example.com
Content-Language: EN
Connection: close
Content-Type: text/html; charset=ISO-8859-1
```

Se il tester ottiene un 405 Method not allowd o un 501 Method Unimplemented, l'app/framework/linguaggio/sistema/firewall funziona correttamente.
Se ottiene una risposta 200, e la risposta non contiene body, è probabile che l'app abbia elaborato la richiesta senza i filtri di autenticazione e di autorizzazione, sono quindi necessari maggiori controlli.

Se il tester pensa che il sistema sia vulnerabile a questo problema, dovrebbe eseguire degli attacchi simili a CSRF per sfruttare questo problema:

- HEAD /admin/createUser.php?member=myAdmin
- HEAD /admin/changePw.php?member=myAdmin&passwd=foo123&confirm=foo123
- JEAD /admin/groupEdit.php?group=Admin&member=myAdmin&action=add

Con un po' di fortuna, usando i tre comandi precedenti - modificati in modo da adattarsi all'app e ai requisiti del test - dovrebbe essere creato un nuovo utente, con una password e di livello amministratore.

## Tools

- netcat
- curl
- script NSE http-methods di nmap