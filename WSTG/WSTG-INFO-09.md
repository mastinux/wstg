# WSTG-INFO-09

# Fingerprint Web Application

## Summary

Data la vastità dei progetti software free o open source distribuiti, è molto probabile che il sito target sia interamente o in parte sviluppato con app note (es. Wordpress, phpBB, Mediawiki, etc.).
Conoscendo i componenti della web app che stiamo testando ci può aiutare nel processo di testing e ridurrà drasticamente lo sforzo richiesto durante i test.
Queste web app note hanno di conseguenza header HTML, cookie, e strutture di directory note che possono essere enumerate per identificare l'app.

## Test Objectives

Identificare la web app e la versione per risalire a vulnerabilità conosciute e scegliere gli adeguati exploit da usare durante i test.

## How to Test

### Cookies

Un modo relativamente affidabile per identificare un'app web è analizzare i cookie specifici dell'app.

Considera la seguente richiesta HTTP:

```
GET / HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 6.2; WOW64; rv:31.0) Gecko/20100101 Firefox/31.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Cookie: wp-settings-time-1=1406093286; wp-settings-time-2=1405988284
DNT: 1
Connection: keep-alive
Host: blog.owasp.org
```

Il cookie wp-settings-time-1 è stato impostato automaticamente e dà informazioni sul framework usato.
La lista di nomi di cookie comuni è presente nella successiva sezione Common Application Identifiers.
Tuttavia, sappiamo che è possibile cambiare il nome del cookie.

### HTML Source Code

Questa tecnica si basa sul cercare dei pattern nel codice sorgente della pagina HTML.
Spesso possiamo trovare molte informazioni che ci aiutano a riconoscere la web app specifica.
Uno degli indicatori comuni sono i commenti HTML che consentono di identificare direttamente l'app.
Molto spesso possono essere trovati alcuni path specifici dell'app, es. i link ai folder di CSS o JS specifici dell'app.
Infine, le variabili di script specifici possono suggerire l'utilizzo di particolari app.

Dal tag meta che segue, possiamo facilmente capire la web app usata dal sito e la sua versione.
Il commento, il path specifico e le variabile degli script possono aiutare l'attaccante a determinare velocemente l'istanza dell'app.

```xml
<meta name="generator" content="WordPress 3.9.2" />
```

Molto spesso queste informazioni sono presenti nei tag `<head>`, nei tag `meta` o alla fine della pagina.
Tuttavia, si raccomanda di controllare l'intero documento dato che può essere utile alla ricerca di commenti utili e di campi nascosti.

### Specific Files and Folders

A parte le informazioni raccolte dalle sorgenti HTML, esiste un altro approccio che aiuta un attaccante a identificare un'app in modo accurato.
Ogni app ha la sua struttura di file e folder specifica sul server.
È stato evidenziato che è possibile vedere i path specifici dal codice sorgente HTML ma a volte non sono presenti esplicitamente nel codice sorgente HTML.

Per scoprirli si può usare una tecnica chiamata dirbusting.
Il dirbusting fa il bruteforcing di un target con nomi di folder e file noti e monitora le risposte HTTP per enumerare i contenuti del server.
Queste informazioni possono essere usate sia per trovare i file di default e attaccarli, che per fare il fingerprint della web app.
Si può usare la funzione Intruder di Burp Suite con una lista di folder noti.

La stessa tecnica può essere usata per fare il dirbusting dei folder di diversi plugin dell'app e della loro versione.

Tip: prima di iniziare il dirbusting, si raccomanda di analizzare il file robots.txt.
A volte possono essere trovati i folder specifici dell'app e altre informazioni sensibili.

Specifici file e folder sono diversi per ogni specifica app.
Si raccomanda di installare l'app corrispondente localmente durante i test per avere una migliore comprensione della sua struttura e di quali file potrebbero essere stati lasciati sul server.
Tuttavia, esistono diverse liste di file noti e un buon esempio è la [wordlist](https://github.com/fuzzdb-project/fuzzdb) di file e folder di FuzzDB.

## Common Application Identifiers

### Cookies

|||
|-|-|
|phpBB|`phpbb3_`|
|Wordpress|`wp-settings`|
|1C-Bitrix|`BITRIX_`|
|AMPcms|`AMP`|
|Django|`CMS django`|
|DotNetNuke|`DotNetNukeAnonymous`|
|e107|`e107_tz`|
|EPiServer|`EPiTrace`, `EPiServer`|
|Graffiti CMS|`graffitibot`|
|Hotaru CMS|`hotaru_mobile`|
|ImpressCMS|`ICMSession`|
|Indico|`MAKACSESSION`|
|InstantCMS|`InstantCMS[logdate]`|
|Kentico CMS|`CMSPreferredCulture`|
|MODx|`SN4[12symb]`|
|TYPO3|`fe_typo_user`|
|Dynamicweb|`Dynamicweb`|
|LEPTON|`lep[some_numeric_value]+sessionid`|
|Wix|`Domain=.wix.com`|
|VIVVO|`VivvoSessionId`|

### HTML Source Code

|Application|Keyword|
|-|-|
|Wordpress|`<meta name="generator" content="WordPress 3.9.2" />`|
|phpBB|`<body id=“phpbb”`|
|Mediawiki|`<meta name="generator" content="MediaWiki 1.21.9" />`|
|Joomla|`<meta name="generator" content="Joomla! - Open Source Content Management" />`|
|Drupal|`<meta name="Generator" content="Drupal 7 (http://drupal.org)" />`|
|DotNetNuke|DNN Platform - http://www.dnnsoftware.com|

## Tools

Di seguito viene presentata una lista di tool generali e noti.
Ci sono anche altre utility, come anche tool di fingerprinting basati su framework particolari.

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