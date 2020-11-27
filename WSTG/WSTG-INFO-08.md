# WSTG-INFO-08

# Fingerprint Web Application Framework

## Summary

Il fingerprint del web framework è un passo importante del processo di information gatering.
Conoscere il tipo di framework può dare un gran vantaggio se tale framework è stato già testato da altri tester.
Non è solo questione di vulnerabilità conosciute nelle versioni non patched ma di specifiche mal configurazioni nel framework e di strutture conosciute di file che rendono il processo di fingerprint così importante.

Sono usati diversi vendor e versioni di web framework.
La possibilità di avere informazioni su di essi aiuta nel processo di testing, e può anche aiutare a cambiare l'evoluzione dei test.
Queste informazioni possono essere ricavate analizzando attentamente alcuni indicatori.
La maggior parte dei framework hanno degli indicatori che li contraddistinguono e che consentono all'attaccante di identificarli.
È ciò che fanno tutti i tool automatici, cercano un indicatore in un posto predefinito e lo confrontano con il database di signature conosciute.
Di solito sono usati più indicatori per ottenere una maggiore accuratezza.

In questa sezione non si fanno differenze tra Web Application Framework e Content Management System (CMS).
È stato fatto per facilitare il fingerprint di entrambi.
Inoltre, ci riferiremo a entrambi con il termine web framework.

## Test Objectives

Definire il tipo di web framework in modo da capire quale metodologia di testing di sicurezza scegliere.

## How to Test

### Black-Box Testing

Ci sono molti posti in cui cercare per determinare il framework usato:

- header HTTP
- cookie
- codice sorgente HTML
- file e folder specifici
- estensioni dei file
- messaggi di errore

### HTTP Headers

Il metodo più semplice per identificare un web framework è analizzare il campo `X-Powered-By` nel header HTTP della risposta.
Molti tool possono essere usati per fare il fingerprint del target.
Il più semplice è `netcat`.

Considera le seguenti richiesta-risposta HTTP:

```
$ nc 127.0.0.1 80
HEAD / HTTP/1.0


HTTP/1.1 200 OK
Server: nginx/1.0.14
Date: Sat, 07 Sep 2013 08:19:15 GMT
Content-Type: text/html;charset=ISO-8859-1
Connection: close
Vary: Accept-Encoding
X-Powered-By: Mono
```

Dal campo `X-Powered-By`, capiamo che il web framework dovrebbe essere Mono.
Comunque, anche se questo approccio è semplice e veloce, questa metodologia non funziona nel 100% dei casi.
È possibile disabilitare facilmente l'header `X-Powered-By` tramite una configurazione appropriata.
Ci sono anche altre tecniche che permettono al sito web di offuscare gli header HTTP (vedi la sezione Remediation).

Quindi nello stesso esempio il tester potrebbe non trovare l'header `X-Powered-By` o ottenere una risposta simile alla seguente:

```
HTTP/1.1 200 OK
Server: nginx/1.0.14
Date: Sat, 07 Sep 2013 08:19:15 GMT
Content-Type: text/html;charset=ISO-8859-1
Connection: close
Vary: Accept-Encoding
X-Powered-By: Blood, sweat and tears
```

A volte ci sono più header HTTP che puntano allo stesso web framework.
Nell'esempio che segue, in base alle informazioni della richiesta HTTP, potremmo vedere che l'header `X-Powered-By` contiene la versione di PHP.
Tuttavia, l'header `X-Generator` afferma che il framework è Swiftlet, che consente all'attaccante di estendere i suoi vettori di attacco.
Durante il fingerprinting, ispeziona attentamente tutti gli header HTTP alla ricerca di informazioni utili.

```
HTTP/1.1 200 OK
Server: nginx/1.4.1
Date: Sat, 07 Sep 2013 09:22:52 GMT
Content-Type: text/html
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/5.4.16-1~dotdeb.1
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
X-Generator: Swiftlet
```

### Cookies

Un altro modo simile e in qualche modo più affidabile è determinare il web framework in base ai cookie.

In base al nome del cookie impostato dalla web app è possibile risalire al framework.
Nella sezione Cookies che trovi a seguire è presente una lista di cookie comuni.
I limiti sono gli stessi - è possibile cambiare il nome del cookie.
Per esempio, per il framework `CakePHP` si può cambiare la configurazione nel seguente modo (ad eccezione di core.php):

```
/**
* The name of CakePHP's session cookie.
*
* Note the guidelines for Session names states: "The session name references
* the session id in cookies and URLs. It should contain only alphanumeric
* characters."
* @link http://php.net/session_name
*/
Configure::write('Session.cookie', 'CAKEPHP');
```

Comunque, questi cambiamenti sono meno probabili rispetto alla modifica all'header `X-Powered-By`, quindi questo approccio può essere considerato più affidabile.

### HTML Source Code

Questa tecnica si basa sulla ricerca di pattern nel codice sorgente della pagina HTML.
Spesso possiamo trovare molte informazioni che possono aiutare un tester a riconoscere uno specifico web framework.
Uno degli indicatori comuni sono i commenti HTML che permettono di individuare direttamente il framework.
Molto spesso possono essere trovati alcuni path specifici del framework, es. i link a cartelle di CSS o di JS specifici del framework.
Infine, variabili di script specifici potrebbero identificare certi framework.

Il commento `<!-- ZK 6.5.1.1 EE 2012121311 -->`, che potremmo trovare nel codice sorgente di una pagina web, indica chiaramente il framework e la sua versione.
Il commento, i path specifici e le variabili degli script possono aiutare l'attacante a determinare rapidamente l'istanza del framework ZK.

Molto spesso tali informazioni si trovano nei tag `<head>`, nei tag `<meta>` o alla fine della pagina.
Tuttavia, è raccomandato controllare tutto il documento dato che può essere utile per altri scopi come la ricerca di altri commenti utili e di campi nascosti.
A volte, agli sviluppatori non interessa nascondere le informazioni del framework usato.
Può essere possibile anche trovare in coda alla pagina `Built upon the Banshee PHP framework v3.1`.

### File Extensions

L'URL potrebbe includere le estensioni dei file.
Le estensioni dei file possono aiutare anche a identificare la piattaforma o la tecnologia web.

Per esempio, OWASP sta usando PHP

`https://www.owasp.org/index.php?title=Fingerprint_Web_Application_Framework&action=edit&section=4`

Ecco alcune estensioni e tecnologie web comuni

- php - PHP
- aspx - Microsoft ASP.NET
- jsp - Java Server Pages

### Error Message

#### Common Frameworks

### Cookies

|Framework|Cookie name|
|-|-|
|Zope|zope3|
|CakePHP|cakephp|
|Kohana|kohanasession|
|Laravel|laravel_session|

### HTML Source Code

#### General Markers

- %framework_name%
- powered by
- built upon
- running

#### Specific Markers

|Framework|Keyword|
|-|-|
|Adobe ColdFusion| `<!-- START headerTags.cfm`|
|Microsoft ASP.NET| `__VIEWSTATE`|
|ZK|`<!-- ZK`|
|Business Catalyst|`<!-- BC_OBNW -->`|
|Indexhibit|`ndxz-studio`|

#### Special Files and Folders

File e folder speciali sono diversi in base allo specifico framework.
È raccomandato installare localmente il framework corrispondente durante i test per avere una comprensione di come è composta l'infrastruttura e di quali file potrebbero essere presenti sul server.
Comunque, esistono già diverse liste di file e un buon esempio è la [wordlist](https://github.com/fuzzdb-project/fuzzdb) di file e folder di FuzzDB.

## Tools

Vediamo una lista di tool.
Ci sono altre utility, come anche tool di fingerprinting basati sui framework.

### WhatWeb

Sito: [https://www.morningstarsecurity.com/research/whatweb](https://www.morningstarsecurity.com/research/whatweb)

Attualmente uno dei migliori tool di fingerprint a disposizione.
È incluso nelle build di default di Kali Linux.
Linguaggio: Ruby.
Le verifiche per il fingerprint sono fatte con:

- string di testo (case sensitive)
- regular expression
- query Google Hack DataBase (insieme di keywork limitato)
- hash MD5
- riconoscimento di URL
- pattern di tag HTML
- codice ruby custom per operazioni passive e aggressive

### BlindElephant

Sito: [http://blindelephant.sourceforge.net/](http://blindelephant.sourceforge.net/)

Questo tool si basa sul principio di verifica dei checksum di file statici specifici delle versioni in modo da fornire un fingerpinting più preciso.
Linguaggio: Python.

### Wappalyzer

Sito: [https://www.wappalyzer.com/](https://www.wappalyzer.com/)

Wappalyzer è un'estensione Firefox/Chrome.
Si basa sulle regular expression e necessita solo di raggiungere la pagina da analizzare nel browser.
Opera interamente a livello di browser e restituisce i risultati in icone.
Anche se a volte ha falsi positivi, è un modo maneggevole per sapere quali tecnologie sono state usate per costruire il sito web subito dopo aver visitato la pagina.

Nota che di default, Wappalyzer invia dati anonimizzati sulla tecnologia in esecuzione sui siti web visitati agli sviluppatori, che vengono poi venduti a terze parti.
Assicurati di disabilitare questo comportamento nelle opzioni dell'estensione.

## Remediation

La raccomandazione è usare i tool descritti prima e controllare i log per capire meglio cosa aiuta un attaccante a individuare il web framework.
Eseguendo più scan dopo aver fatto delle modifiche per nascondere le tracce del framework, è possibile ottenere un miglior livello di sicurezza e assicurarsi che il framework non possa essere identificato da scan automatici.
Seguono alcune raccomandazioni specifiche.

### HTTP Headers

Controlla la configurazione e disabilita oppure offusca tutti gli header HTTP che rivelano informazioni sulle tecnologie usate.

### Cookies

Si raccomanda di cambiare i nomi dei cookie eseguendo la modifica negli appositi file di configurazione.

### HTML Source Code

Controlla manualmente il contenuto del codice HTML e rimuovi tutto ciò che rimanda specificatamente al framework.

Linee guida generali:

- assicurati che non ci siano indicatori visivi che possano rivelare il framework
- rimuovi tutti i commenti non necessari (copyright, informazioni su bug, commenti specifici del framework)
- rimuovi i tag META e generator
- usa i file CSS o JS dell'azienda e non memorizzarli in un folder specifico del framework
- non usare gli script di default nella pagina oppure offuscali prima di usarli

### Specific Files and Folders

Linee guida generali:

- rimuovi qualsiasi file non necessario o non usato dal server.
Sono inclusi i file di testo che contengono informazioni su versioni e installazioni
- restringi l'accesso agli altri file per restituire un 404 quando si prova ad accedere dall'esterno.
Si può per esempio modificare il file htaccess aggiungendo RewriteCond o RewriteRule.
Segue un esempio di questo tipo di restrizioni di due folder WordPress comuni.

```
RewriteCond %{REQUEST_URI} /wp-login\.php$ [OR]
RewriteCond %{REQUEST_URI} /wp-admin/$
RewriteRule $ /http://your_website [R=404,L]
```

Comunque, non sono gli unici modi per restringere l'accesso.
Per automatizzare questo processo, esistono dei plugin specifici del framework.
Un esempio per WorkPress è StealthLogin.

### Additional Approaches

Linee guida generali:

- checksum management: 
lo scopo di questo approccio è di contrastare gli scanner basati su checksum impedendo loro di individuare i file tramite i loro hash.
Di solito, ci sono due approcci nel checksum management:
	- cambiare il posto in cui si trovano questi file (es. spostarli in un altro folder, o rinominare il folder che li contiene)
	- modificare il contenuto - una leggera modifica genera un hash completamente diverso, allora l'agginta di un singolo byte alla fine del file non dovrebbe essere un grosso problema
- controlled chaos:
un metodo divertente ed efficace che consiste nell'aggiungere file fasulli di altri framework per confondere gli scanner e l'attaccante.
Ma fai attenzione a non sovrascrivere i file e i folder buoni per non rompere il framework attuale