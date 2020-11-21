# Fingerprint Web Server

## Summary

Il fingerprinting del web server punta a identificare il tipo e la versione del web server su cui il target è in esecuzione.
Il fingerprinting del web server è spesso incorporato in tool di testing automatizzati, ma è utile ai ricercatori per capire i principi secondo cui i tool provano a identificare il software e perchè sia utile.

L'identificazione puntuale del tipo di web server su cui un'applicazione è in esecuzione può consentire ai tester di determinare se l'applicazione è vulnerabile agli attacchi.
In particolare, i server che usano versioni più vecchie di software senza patch di sicurezza possono essere suscettibili a exploit conosciuti specifici della versione.

## Test Objectives

Determinare la versione e il tipo di server in esecuzione per consentire l'individuazione di vulnerabilità conosciute.

## How to Test

Le tecniche usate per il fingerprint di web server includono
il banner grabbing,
l'invio di richieste malformate,
e l'uso di tool automatici per eseguire scan più robusti che usano una combinazione di probe.
La premessa fondamentale su cui tutte le precedenti tecniche si basano è la stessa.
Provano a ottenere una risposta dal web server che può poi essere confrontata con un database di risposte e di comportamenti conosciute, e quindi che corrispondono a un tipo di server conosciuto.

### Banner Grabbing

Il banner grabbing è eseguito inviando una richiesta HTTP al web server ed esaminando gli header della risposta.
Puoi usare diversi tool, tra cui `telnet` per richieste HTTP, o `openssl` per richieste su SSL.

Per esempio, questa è la risposta a una richiesta a un server Apache.

```
HTTP/1.1 200 OK
Date: Thu, 05 Sep 2019 17:42:39 GMT
Server: Apache/2.4.41 (Unix)
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
ETag: "75-591d1d21b6167"
Accept-Ranges: bytes
Content-Length: 117
Connection: close
Content-Type: text/html
...
```

Segue un'altra risposta, questa volta da nginx.

```
HTTP/1.1 200 OK
Server: nginx/1.17.3
Date: Thu, 05 Sep 2019 17:50:24 GMT
Content-Type: text/html
Content-Length: 117
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Connection: close
ETag: "5d71489a-75"
Accept-Ranges: bytes
...
```

Ecco una risposta da un lighttpd.

```
HTTP/1.0 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4192788355"
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Content-Length: 117
Connection: close
Date: Thu, 05 Sep 2019 17:57:57 GMT
Server: lighttpd/1.4.54
```

In questi esempi, il tipo e la versione del server sono esposti.
Tuttavia, le app security-conscious potrebbero offuscare le informazioni sul proprio server modificando l'header.
Per esempio, ecco un estratto della risposta a una richiesta a un sito con header modificato:

```
HTTP/1.1 200 OK
Server: Website.com
Date: Thu, 05 Sep 2019 17:57:06 GMT
Content-Type: text/html; charset=utf-8
Status: 200 OK
...
```

Quando le informazioni del server sono offuscate, i tester potrebbero indovinare il tipo di server in base all'ordine degli header.
Nota che nell'esempio precedente di Apache, i campi seguono questo ordine:

- Date
- Server
- Last-Modified
- ETag
- Accept-Ranges
- Content-Length
- Connection
- Content-Type

Tuttavia, sia negli esempi di server nginx e offuscati, i campi in comune seguono questo ordine:

- Server
- Date
- Content-Type

I tester possono usare queste informazioni per indovinare se il server offuscato è nginx.
Comunque, considerando che diversi web server potrebbero condividere lo stesso ordine e i campi possono essere modificati o rimossi, questo metodo non è particolarmente preciso.

### Sending Malformed Requests

I web server potrebbero essere identificati esaminando i loro messaggi di errore e le loro pagine di errore di default, nel caso in cui queste non siano state customized.
Un modo per costringere il server a presentare queste pagine è inviare intenzionalmente richieste non corrette o malformate.

Per esempio, ecco un esempio di una richiesta per il metodo HTTP non esistente `SANTA CLAUS` verso un server Apache.

```
GET / SANTA CLAUS/1.1


HTTP/1.1 400 Bad Request
Date: Fri, 06 Sep 2019 19:21:01 GMT
Server: Apache/2.4.41 (Unix)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
```

Ecco la risposta per la stessa richiesta da nginx.

```
GET / SANTA CLAUS/1.1


<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.17.3</center>
</body>
</html>
```

Ecco la risposta alla stessa richiesta da lighttpd.

```
GET / SANTA CLAUS/1.1


HTTP/1.0 400 Bad Request
Content-Type: text/html
Content-Length: 345
Connection: close
Date: Sun, 08 Sep 2019 21:56:17 GMT
Server: lighttpd/1.4.54

<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
<title>400 Bad Request</title>
</head>
<body>
<h1>400 Bad Request</h1>
</body>
</html>
```

Dato che le pagine di errore di default offrono diversi fattori di differenziazione tra tipi di web server, la loro analisi può essere un metodo efficace per il fingerprinting anche quando gli header del server sono offuscati.

### Using Automated Scanning Tools

Come indicato prima, il fingerprinting di web server è spesso incluso come funzionalità dei tool di scanning automatizzato.
Questi tool sono in grado di fare richieste simili a quelle mostrate prima, come anche l'invio di altri probe specifici per il server.
I tool automatizzati possono confrontare risposte da web server più velocemente rispetto al testing manuale, e usano grandi database di risposte conosciute per provare a identificare il server.
Per queste ragioni, i tool automatizzati dovrebbero produrre risultati più accurati.

Ecco alcuni tool di scan comunemente usati che includono le funzionalità di fingerprinting di web server.

- netcraft
- nikto
- nmap

## Remediation

Anche se l'informazione del server esposta non è di per sè una vulnerabilità, può aiutare gli attaccanti a trovare altre vulnerabilità che potrebbero esistere.
Le informazioni del server esposte possono guidare gli attaccanti a trovare vulnerabilità del server specifiche della versione che possono essere usate per l'exploiting di server non patched.
Per queste ragioni si raccomanda di prendere alcune precauzioni.
Queste includono:

- offuscare le informazioni sul web server negli header
- usare un reverse proxy hardened per creare un layer di sicurezza aggiuntivo tra il web server e Internet
- assicurarsi che il web server sia aggiornato all'ultima versione e con le ultime patch di sicurezza