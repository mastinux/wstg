# WSTG-ATHN-01

# Testing for Credentials Transported over an Encrypted Channel

## Summary

Testare il trasporto delle credenziali significa verificare che i dati di autenticazione dell'utente siano trasferiti su un canale cifrato per evitare che siano intercettati da utenti malevoli.
L'analisi si focalizza semplicemente sul capire se i dati viaggiano senza cifratura dal web browser al server, o se la web app adotta le adeguate misure di sicurezza nell'uso di protocolli come HTTPS.
Il protocollo HTTPS si basa su TLS/SSL per cifrare i dati che sono trasferiti e per assicurare che l'utente sia indirizzato verso il sito adeguato.

Chiaramente, il fatto che il traffico sia cifrato non significa necessariamente che sia completamente sicuro.
La sicurezza dipende anche sull'algoritmo di cifratura usato e dalla robustezza delle chiavi che l'app usa, ma quest'argomento non verrà trattato in questa sezione.

Per una discussione più dettagliata riguardo il testing sulla sicurezza dei canali TLS/SSL fai riferimento al capitolo [Testing for Weak SSL TLS Ciphers Insufficient Transport Layer Protection](./WSTG/WSTG-CRYP-01.md).
Qui, il tester cercherà di capire se i dati che l'utente inserisce nei web form per accedere ai siti, sono trasmessi usando protocolli sicuri in modo da proteggerlo dall'attaccante.

Al giorno d'oggi, l'esempio più diffuso è la pagina di login di una web app.
Il tester dovrebbe verificare che le credenziali dell'utente siano trasmesse tramite un canale cifrato.
Per accedere a un sito web, di solito l'utente deve compilare un semplice form che trasmette i dati inseriti a una web app con il metodo POST.
Ciò che è meno ovvio è che questi dati siano trasmessi tramite il protocollo HTTP, che 
li trasmette in forma non sicura, in clear text, 
o usando il protocollo HTTPS, che cifra i dati durante la trasmissione.
Per complicare ulteriormente la situazione, è possibile che il sito abbia la pagina di login accessibile tramite HTTP, ma poi effettivamente invii i dati su HTTPS.
Questo test viene eseguito per assicurarsi che l'attaccante non possa recuperare informazioni sensibili semplicemente facendo lo sniffing del traffico di rete con uno sniffer tool.

## How to Test

### Black-Box Testing

Nei seguenti esempi useremo un web proxy per catturare gli header e ispezionarli.
Puoi usare qualsiasi web proxy.

#### Example 1: Sending Data with POST Method Through HTTP

Supponi che la pagina di login presenti un form con i campi User, Pass, e il button Submit per l'autenticazione e che dia accesso all'app.
Possiamo dare un'occhiata agli header della richiesta con un web proxy:

```
POST http://www.example.com/AuthenticationServlet HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it; rv:1.8.1.14) Gecko/20080404
Accept: text/xml,application/xml,application/xhtml+xml
Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://www.example.com/index.jsp
Cookie: JSESSIONID=LVrRRQQXgwyWpW7QMnS49vtW1yBdqn98CGlkP4jTvVCGdyPkmn3S!
Content-Type: application/x-www-form-urlencoded
Content-length: 64

delegated_service=218&User=test&Pass=test&Submit=SUBMIT
```

Da questo esempio il tester può capire che la richiesta POST invia dati alla pagina `http://www.example.com/AuthenticationServlet` usando HTTP.
Quindi i dati sono trasmessi senza cifratura e un utente malevolo potrebbe intercettare username e password usando uno sniffing tool come Wireshark.

#### Example 2: Sending Data with POST Method Through HTTPS

Supponi che la nostra web app usi il protocollo HTTPS per cifrare i dati che stiamo inviando (o almeno per la trasmissione di dati sensibili come le credenziali).
In questo caso, quando si accede alla web app gli header della nostra richiesta POST potrebbero essere simili ai seguenti:

```
POST https://www.example.com:443/cgi-bin/login.cgi HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it; rv:1.8.1.14) Gecko/20080404
Accept: text/xml,application/xml,application/xhtml+xml,text/html
Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: https://www.example.com/cgi-bin/login.cgi
Cookie: language=English;
Content-Type: application/x-www-form-urlencoded
Content-length: 50

Command=Login&User=test&Pass=test
```

Possiamo vedere che la richiesta è indirizzata a `https://www.example.com:443/cgi-bin/login.cgi` tramite il protocollo HTTPS.
Ciò assicura che le nostre credenziali siano inviate usando un canale cifrato e che le credenziali non siano leggibili da un utente malevolo tramite uno sniffer.

#### Example 3: Sending Data with POST Method via HTTPS on a Page Reachable via HTTP

Ora, immagina di avere una pagina web raggiungibile tramite HTTP e che solo i dati inviati dal form di autenticazione siano trasmessi su HTTPS.
Questa situazione si verifica, per esempio, quando siamo su un portale di una grossa azienda che offre varie informazioni e servizi che sono accessibili pubblicamente, senza identificazione, ma il sito ha anche una sezione privata accessibile dalla home page dopo che l'utente ha fatto l'accesso.
Quando eseguiamo il login, gli header della nostra richiesta saranno simili ai seguenti:

```
POST https://www.example.com:443/login.do HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it; rv:1.8.1.14) Gecko/20080404
Accept: text/xml,application/xml,application/xhtml+xml,text/html
Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://www.example.com/homepage.do
Cookie: SERVTIMSESSIONID=s2JyLkvDJ9ZhX3yr5BJ3DFLkdphH0QNSJ3VQB6pLhjkW6F
Content-Type: application/x-www-form-urlencoded
Content-length: 45

User=test&Pass=test&portal=ExamplePortal
```

Possiamo vedere che la nostra richiesta è indirizzata a `www.example.com:443/login.do` usando HTTPS.
Ma se diamo un'occhiata all'header `Referer` (la pagina da cui proviene la richiesta), è `www.example.com/homepage.do` ed è accessibile tramite il semplice HTTP.
Anche se stiamo inviando i dati tramite HTTPS, questa configurazione può essere suscettibile agli attacchi SSLStrip.

> L'attacco precedente è un attacco Man-in-the-middle.

#### Example 4: Sending Data with GET Method Through HTTPS

In questo ultimo esempio, supponi che l'app trasporti i dati usando il metodo GET.
Questo metodo non dovrebbe mai essere usato in un form per trasmettere dati come username e password, dato che i dati sono mostrati in clear text nell'URL e genera una serie di problemi di sicurezza.
Per esempio, l'URL richiesta è facilmente accessibile dai log del server o dall'history del browser.
Quindi questo esempio è puramente dimostrativo, invece è fortemente raccomandato l'uso del metodo POST.

```
GET https://www.example.com/success.html?user=test&pass=test HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; it; rv:1.8.1.14) Gecko/20080404
Accept: text/xml,application/xml,application/xhtml+xml,text/html
Accept-Language: it-it,it;q=0.8,en-us;q=0.5,en;q=0.3
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: https://www.example.com/form.html
If-Modified-Since: Mon, 30 Jun 2008 07:55:11 GMT
If-None-Match: "43a01-5b-4868915f"
```

Poi vedere che i dati sono trasmessi in clear text nell'URL e non nel body della richiesta come prima.
Ma dobbiamo considerare che SSL/TLS è un protocollo di livello 5, un livello più in basso di HTTP, quindi l'intero pacchetto HTTP è cifrato e l'URL non è leggibile all'utente malevolo che usa uno sniffer.
Tuttavia come detto prima, non è una good practice usare il metodo GET per inviare dati sensibili a una web app, perchè le informazioni contenute nell'URL possono essere memorizzate in diversi posti come nei log del proxy o del web server.

### Gray-Box Testing

Parla con gli sviluppatori della web app e prova a capire se sono coscienti delle differenze tra i protocolli HTTP e HTTPS e perchè dovrebbero usare HTTPS per trasmettere informazioni sensibili.
Poi, verifica insieme a loro se viene usato HTTPS per ogni richiesta contenente dati sensibili, come quella della pagina di login, per impedire a utenti non autorizzati di intercettarli.

## Tools

- OWASP Zed Attack Proxy
- mitmproxy
- Burp Suite
- Wireshark
- TCPDUMP