# WSTG-INFO-04

# Enumerate Applications on Webserver

## Summary

Un passo fondamentale nel testing di vulnerabilità delle web app è individuare quali app sono ospitate sul web server.
Molte app hanno vulnerabilità conosciute e strategie di attacco conosciute che possono essere sfruttate per ottenere un controllo remoto o estrarre dati.
Inoltre, molte app sono spesso mal configurate o non aggiornate, presupponendo che sono usate solo "internamente" e quindi non ci sono minacce.
Con la proliferazione di web server virtuali, la tradizionale relazione 1:1 tra un indirizzo IP e un web server sta perdendo il suo significato originale.
È molto comune trovare più siti web o app il cui nome simbolico viene risolto con lo stesso indirizzo IP.
Questo scenario non è limitato agli ambienti di hosting, ma anche agli ambienti aziendali.

Ai tester viene spesso dato un insieme di indirizzi IP come target del test.
In questo caso, si andrebbero a testare tutte le web app accedibili tramite questi target.
Il problema è che il dato indirizzo IP ospita un servizio HTTP sulla porta 80, ma se un tester ci accede specificando il solo indirizzo IP (che è quello che conosce), il server risponderà con "No web server configured at this address" o un messaggio simile.
Ma quel sistema potrebbe "nascondere" diverse web app, associate a nomi di dominio non correlati.
Logicamente, l'analisi è fortemente influenzata dai test eseguiti su tutte le app o solo su quelle che il tester conosce.

A volte, l'indicazione del target è più accurata.
Al tester potrebbero essere indicati gli indirizzi IP e i corrispondenti nomi di dominio.
Tuttavia, questa lista potrebbe non essere completa, 
es. potrebbe omettere alcuni nomi di dominio e il cliente potrebbe non esserne a conoscenza (molto probabile in grandi organizzazioni).

Altri problemi che riguardano lo scope dell'assessment sono le web app pubblicate su URL non ovvie (es. `http://www.example.com/some-strange-URL`), che non sono referenziate da altre pagine.
Potrebbe verificarsi per errore (a causa di una configurazione errata), o intenzionalmente (es. interfacce amministrative).

Per risolvere questi problemi, è necessario eseguire una web app discovery.

## Test Objectives

Enumerare le applicazioni nello scope del web server.

## How to Test

### Black-Box Testing

Il discovery di web app è un processo che mira a identificare le web app ospitate in una data infrastruttura.
Quest'ultima è di solito specificata come un insieme di indirizzi IP (potrebbe anche essere una sotto-rete), ma potrebbe consistere in un insieme di nomi di dominio o un mix dei precedenti.
Queste informazioni sono comunicate prima dell'esecuzione dell'assessment, sia in caso di un penetration testing classico che un assessment focalizzato sull'app.
In entrambe i casi, a meno che le Rules of Engagement (RoE) specifichino diversamente (es. testare solo l'app ospitata su `http://www.example.com`), l'assessment dovrebbe sforzarsi di essere quanto più possibile completo nello scope, es. dovrebbe identificare tutte le app accedibili tramite il dato target.
Il seguente esempio esamina alcune tecniche che possono essere adottate per ottenere questo risultato.

> Alcune delle seguenti tecniche si applicano ai web server affacciati su Internet, cioè servizi di ricerca DNS  e reverse-IP web-based e l'uso dei motori di ricerca.
> Gli esempi usano indirizzi IP privati (come `192.168.1.100`), che, a meno che diversamente indicato, rappresentano indirizzi IP generici e sono usati solo a scopo di anonimato.

Questi sono tre fattori che influenzano il fatto che molte app siano relazionate a un dato nome di dominio (o un indirizzo IP):

1. Different Base URL

L'entry point ovvio di una web app è `www.example.com`, es. con questa abbreviazione pensiamo alla web app attiva su `http://www.example.com/` (lo stesso vale per https).
Comunque, anche se questa è la situazione più comune, non c'è nulla che forza l'app a partire da `/`.

Per esempio, lo stesso nome simbolico potrebbe essere associato a tre web app come: 
`http:/www.example.com/url1` 
`http:/www.example.com/url2` 
`http:/www.example.com/url3`

In questo caso, l'URL `http://www.example.com/` potrebbe non essere associata ad alcuna pagina, e le tre app potrebbero essere "nascoste", a meno che il tester non sappia come raggiungerle, es. il tester conosce url1, url2 e url3.
Di solito non c'è motivo di pubblicare le web app in questo modo, a meno che il proprietario non vuole che vengano accedute nel modo standard, e sia in grado di informare gli utenti sulla loro posizione esatta.
Ciò non significa che le app siano segrete, solo che la loro esistenza e la loro posizione non sono pubblicate esplicitamente.

2. Non-standard Ports

Anche se le web app di solito rispondono sulla porta 80 (http) e 443 (https), non è un obbligo.
Infatti, le web app potrebbero essere associate a porte TCP arbitrarie, e possono essere referenziate specificando il numero di porta come segue: `http\[s\]://www.example.com:port/`. 
Per esempio, `http://www.example.com:20000/`.

3. Virtual Hosts

Il DNS permette a un singolo indirizzo IP di essere associato con uno o più nomi simbolici.
Per esempio, l'indirizzo IP `192.168.1.100` potrebbe essere associato ai nomi DNS 
`www.example.com`, 
`helpdesk.example.com`, 
`webmail.example.com`.
Non è necessario che tutti i nomi appartengano allo stesso dominio DNS.
Questa relazione 1:N potrebbe essere usata per usare i virtual host.
L'informazione che specifica il virtual host a cui ci vogliamo agganciare è specificata nell'header `Host` di HTTP 1.1.

Potresti non sospettare l'esistenza di altre web app oltre all'ovvia `www.example.com` a meno che non tu conosca `helpdesk.example.com` e `webmail.example.com`.

#### Approaches to Address Issue 1 - Non-standard URLs

Non c'è modo di accertarsi completamente dell'esistenza di web app che non hanno un nome standard.
Non essendo standard, non esiste un criterio che regoli la convenzione di nomenclatura, tuttavia ci sono alcune tecniche che il tester può usare per ottenere più informazioni.

Primo, se il web server è mal configurato e consente il directory browsing, potrebbe essere possibile individuare queste app.
I vulnerability scanner potrebbero essere d'aiuto.

Secondo, queste app potrebbero essere referenziate da altre pagine web ed esiste la possibilità che siano state indicizzate dai motori di ricerca.
Se i tester sospettano l'esistenza di queste app "nascoste" su `www.example.com` potrebbero cercarle usando l'operator `site` ed esaminare il risultato della query `site:www.example.com`.
Tra le URL restituite potrebbe essercene una che punta all'app "nascosta".

Un'altra opzione è accedere alle URL che potrebbero essere candidate ad essere app non pubblicate.
Per esempio, un front end per web mail potrebbe essere associato alle URL 
`https://www.example.com/webmail`, 
`https://webmail.example.com/` o 
`https://mail.example.com/`.
Lo stesso vale per le interfacce amministrative, che possono essere pubblicate su URL nascoste (per esempio, un'interfaccia amministativa di Tomcat), e non essere referenziate altrove.
Quindi fare un po' di ricerca basata su dizionario (o "intelligent guessing") potrebbe portare dei risultati.
I vulnerability scanner potrebbero essere d'aiuto.

#### Approaches to Address Issue 2 - Non-standard Ports

È facile verificare l'esistenza di web app su port non standard.
Un port scanner come nmap è in grado di eseguire la service recognition tramite l'opzione `-sV`, e identificare i servizi http[s] su porte arbitrarie.
È necessario uno scan completo di tutte le 64k porte.

Per esempio, il seguente comando controllerà, con un TCP connect scan, tutte le porte aperte sull'IP `192.168.1.100` e proverà a determinare quali servizi sono attivi su di esse (vedremo solo gli switch principali - nmap supporta un vasto numero di opzioni, la cui discussione è out of scope)

`nmap –Pn –sT –sV –p0-65535 192.168.1.100`

È sufficiente per esaminare l'output e per cercare http o l'indicatore di servizi su SSL (che dovrebbero essere verificati separatamente per confermare che siano https).
Per esempio, l'output del precedente comando potrebbe essere:

```sh
Interesting ports on 192.168.1.100:
(The 65527 ports scanned but not shown below are in state: closed)
PORT 		STATE 		SERVICE VERSION
22/tcp 		open 		ssh 	OpenSSH 3.5p1 (protocol 1.99)
80/tcp 		open 		http 	Apache httpd 2.0.40 ((Red Hat Linux))
443/tcp 	open ssl 	OpenSSL
901/tcp 	open http 	Samba SWAT administration server
1241/tcp 	open ssl 	Nessus security scanner
3690/tcp 	open unknown
8000/tcp 	open http-alt?
8080/tcp 	open http 	Apache Tomcat/Coyote JSP engine 1.1
```

Da questo esempio, possiamo vedere:

- un server HTTP Apache è in esecuzione sulla porta 80
- sembra che ci sia un server HTTPS sulla porta 443 (ma deve essere confermato, per esempio, visitando `https://192.168.1.100` col browser)
- sulla porta 901 è attiva un'interfaccia web Samba SWAT
- il servizio sulla porta 1241 non è HTTPS, ma è il daemon Nessus su SSL
- la porta 3690 ospita un servizio non specificato (nmap restituisce il suo fingerprint - qui omesso per chiarezza - insieme alle istruzioni da usare per incorporare il suo fingerprint nel database di nmap, assunto che tu conosca il servizio esposto)
- un altro servizio non usato sulla porta 8000; potrebbe essere HTTP, dato che non è raro trovare server HTTP su questa porta.
Verifichiamolo:

```sh
$ telnet 192.168.10.100 8000
Trying 192.168.1.100...
Connected to 192.168.1.100.
Escape character is '^]'.
GET / HTTP/1.0


HTTP/1.0 200 OK
pragma: no-cache
Content-Type: text/html
Server: MX4J-HTTPD/1.0
expires: now
Cache-Control: no-cache

<html
```

Ciò conferma il fatto che è un server HTTP.
Alternativamente, i tester potrebbero visitare l'URL con un browser; 
o usare i comandi Perl GET o HEAD, che simula le interazioni HTTP come quelle viste prima (tuttavia le richieste HEAD potrebbero non essere accettate da tutti i server).

- Apache Tomcat in esecuzione sulla porta 8080

Lo stesso compito può essere eseguito dai vulnerability scanner, ma prima verifica che lo scanner che hai scelto sia in grado di identificare i servizi HTTP[S] in esecuzione sulle porte non standard.
Per esempio, Nessus è in grado di identificarli su porte arbitrarie (presupposto che sia configurato per scansionare tutte le porte), e fornirà un numero di test di vulnerabilità server conosciute, come tutte le configurazioni SSL dei servizi HTTPS.
Come accennato prima, Nessus è anche in grado di individuare web app o interfacce web che potrebbero non essere individuate diversamente (per esempio, un'interfaccia amministrativa di Tomcat).

#### Approaches to Address Issue 3 - Virtual Hosts

Ci sono diverse tecniche che potrebbero essere usate per identificare i nomi DNS associati con un dato indirizzo IP `x.y.z.t`.

##### DNS Zone Transfers

Al giorno d'oggi questa tecnica ha un uso limitato, dato che i zone transfer non sono in gran parte onorati dai server DNS.
Tuttavia, potrebbe essere utile provarci.
Prima di tutto, i tester devono determinare i name server che servono `x.y.z.t`.
Se un nome simbolico è conosciuto per `x.y.z.t` (consideriamo `www.example.com`), i suoi name server possono essere individuati tramite tool come `nslookup`, `host` o `dig`, richiedendo i record DNS NS.

Se non sono conosciuti nomi simbolici per `x.y.z.t`, ma il target conosciuto contiene almeno un nome simbolico, i tester possono provare ad applicare lo stesso processo e interrogare lo stesso name server di quel nome (sperando che `x.y.z.t` sarà servito da quel name server).
Per esempio, se il target è composto dall'indirizzo IP `x.y.z.t` e il nome `www.example.com`, determina i name server per il dominio `example.com`.

Il seguente esempio mostra come identificare i name server per `www.owasp.org` usando il comando `host`:

```sh
$ host -t ns www.owasp.org
www.owasp.org is an alias for owasp.org.
owasp.org name server ns1.secure.net.
owasp.org name server ns2.secure.net.
```

Potrebbe non essere richiesto uno zone transfer al name server per il dominio `example.com`.
Se il tester è fortunato, otterrà una lista di entry DNS per questo dominio.
Includerà `www.example.com` e i non ovvi `helpdesk.example.com` e `webmal.example.com` (e possibilmente altri).
Controlla tutti i nomi ricevuti dallo zone transfer e considera tutti quelli in relazione con il target.

Prova a richiedere uno zone transfer per `owasp.org` da uno dei suoi name server:

```sh
$ host -l www.owasp.org ns1.secure.net
Using domain server:
Name: ns1.secure.net
Address: 192.220.124.10#53
Aliases:

Host www.owasp.org not found: 5(REFUSED)
; Transfer failed.
```

##### DNS Inverse Queries

Questo processo è simile al precedente, ma si basa sui record DNS inversi (PTR).
Piuttosto che richiedere uno zone transfer, prova a impostare il tipo di record a PTR e invia una richiesta per il dato indirizzo IP.
Se il tester è fortunato, potrebbe ricevere un entry DNS.
Questa tecnica si basa sull'esistenza di mappe IP-to-symbolic name, che però non è garantita.

##### Web-based DNS Searches

Questo tipo di ricerca è simile allo zone transfer DNS, ma si basa su servizi web-based che consentono ricerche basate sul nome nel DNS.
Un servizio del genere è il servizio Netcraft Search DNS.
Il tester potrebbe cercare una lista di nomi appartenenti a un dominio a scelta, come `example.com`.
Poi controllerà se i nomi ottenuti sono pertinenti al target sotto esame.

##### Reverse-IP Services

I servizi di reverse-IP sono simili alle inverse query DNS, 
con la differenza che il tester interroga un'app web-based piuttosto che un name server.
Ci sono diversi servizi del genere disponibili.
Dato che restituiscono risultati parziali (o spesso diversi), è meglio usare servizi diversi per ottenere un'analisi più completa.

- Domain Tools Reverse IP
- Bing: sintassi `ip:x.x.x.x`
- Webhosting Info: sintassi: `http://whois.webhosting.info/x.x.x.x`
- DNSstuff
- Net Square

##### Googling

A seguito dell'information gatering realizzato con le precedenti tecniche, i tester possono usare i motori di ricerca per perfezionare e incrementare il risultato della loro analisi.
Potrebbero ottenere altri nomi simbolici appartenenti al target, o alle app accessibili tramite le URL non ovvie.

Per esempio, considerando il precedente esempio per `www.owasp.org`, 
il tester potrebbe interrogare Google o altri motori di ricerca per avere informazioni (quindi, nomi DNS) relative ai nuovi domini scoperti di `webgoat.org`, `webscarab.com` e `webscarab.net`.

Queste tecniche sono spiegate in [Conduct Search Engine Discovery Reconnaissance for Information Leakage](./WSTG-INFO-01.md)

### Grey-Box Testing

Non applicabile.
La metodologia rimane la stessa come mostrato nel black-box testing indipendentemente da quali sono le informazioni di partenza.

## Tools

- lookup tools DNS come `nslookup`, `dig` e simili
- motori di ricerca
- servizi di ricerca web-based specifici per DNS
- nmap
- Nessuss Vulnerability Scanner
- Nikto