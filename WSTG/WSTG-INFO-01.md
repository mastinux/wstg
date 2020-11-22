# WSTG-INFO-01 

# Conduct Search Engine Discovery Reconnaissance for Information Leakage

## Summary

Per permettere ai motori di ricerca di funzionare, i loro programmi (o "robot") recuperano dati (fanno crawling) da miliardi di pagine nel web.
Questi robot trovano le pagine web seguendo i link da altre pagine, o analizzando le sitemap.
Un sito web usando un file speciale chiamato "robots.txt" elenca le pagine che non vuole che vengano analizzate dai motori di ricerca.
Questa è una panoramica di base - Google offre una spiegazione più approfondita su come funziona un motore di ricerca.

I tester possono usare i motori di ricerca per fare la reconnaissance sui siti web e sulle web app.
Ci sono due metodi per usare i motori di ricerca:
i metodi diretti consistono nel cercare gli indici e il contenuto tenuto in cache,
mentre i metodi indiretti consistono nel recuperare informazioni sensibili su design e configurazione cercando nei forum, nei newsgroup e siti web di tendering.

Dopo che un motore di ricerca ha terminato il crawling, fa l'indexing della pagina web sulla base dei tag e degli attributi associati, come `<title>`, per restituire i risultati di ricerca rilevanti.
Se il file robots.txt non è stato caricato, e i meta tag HTML che indicano ai robot di non fare l'indexing del contenuto non sono stati usati, è possibile che degli indici mantengano il contenuto web che i proprietari non volevano includere.
I proprietari del sito web potrebbero usare il file robots.txt, i meta tag HTML, l'autenticazione, e i tool forniti dai motori di ricerca per rimuovere questo contenuto.

## Test Objectives

Capire quali informazioni sensibili su design e configurazione dell'applicazione, del sistema, o dell'organizzazione sono esposte sia direttamente (nel sito web dell'organizzazione) o indirettamente (su siti web di terze parti).

## How to Test

Usa un motore di ricerca per cercare informazioni potenzialmente sensibili.
Queste potrebbero essere:

- diagrammi e configurazioni di rete;
- post ed email archiviati da amministratori o da altro staff;
- log su procedure e formato dell'username;
- username, password, e chiavi private;
- file di configurazione di terze parti, o servizi cloud
- contenuti che rivelano messaggi di errore
- versioni di sviluppo, di test, di User Acceptance Testing (UAT), e di staging del sito web

### Search Engines

Non limitare la ricerca a un solo motore di ricerca, dato che diversi motori di ricerca possono generare diversi risultati.
I risultati dei motori di ricerca possono variare di poco, in base a quando il motore ha fatto il crawling, e all'algoritmo usato dal motore per determinare se le pagine sono rilevanti.
È consigliato usare i seguenti motori di ricerca:

- Baidu
- Bing
- bindsearch.info
- DuckDuckGo
- Google
- Smartpage
- Shodan

Sia DuckDuckGo che Startpage offrono una migliore privacy agli utenti non utilizzando i tracker e non memorizzando i log.
In questo modo si può ridurre l'information leakage nei confronti del tester.

### Search Operators

Un search operator è una keyword speciale che estende le capacità di una ricerca normale, e può aiutare a ottenere risultati più specifici.
In generale sono nel formato `operator:query`.
Di seguito sono elencati i search operator più comunemente supportati:

- `site`: limita la ricerca nell'URL fornita
- `inurl`: restituisce solo i risultati che includono la keyword nell'URL
- `intitle`: restituisce solo i risultati che includono la keyword nel title della pagina
- `intext` o `inbody`: cercherà la keyword solo nel body delle pagine
- `filetype`: cercherà solo uno specifico tipo di file

Per esempio, per trovare il contenuto web di owasp.org indicizzato secondo il motore di ricerca, la sintassi richiesta è `site:owasp.org`.

### Viewing Cached Content

Per cercare il contenuto che è stato precedentemente indicizzato, usa il search operator `cache:`.
È utile per visualizzare il contenuto che potrebbe essere cambiato da quando è stato indicizzato, o potrebbe non essere più accedibile.
Non tutti i motori di ricerca forniscono i contenuto tenuti in cache; la risorsa più utile è attualmente Google.
Per vedere il contenuto di owasp.org tenuto in cache, la sintassi è `cache:owasp.org`.

### Google Hacking, or Dorking

La ricerca con gli operator può essere una tecnica di discovery reconnaissance molto efficace se combinata con la creatività del tester.
I search operator possono essere concatenati per scoprire file e informazioni sensibili specifici.
Questa tecnica, chiamata Google hacking o Google dorking, può essere usata anche con altro motori di ricerca, se i relativi search operator sono supportati.

Un database di dork, come Google Hacking Database, è una risorsa utile che può aiutare a individuare informazioni specifiche.
Alcune categorie di dork disponibili in questo database includono:

- Footholds
- Files containing usernames
- Sensitive Directories
- Web Server Detection
- Vulnerable Files
- Vulnerable Servers
- Error Messages
- Files containing juicy info
- Files containing passwords
- Sensitive Online Shopping Info

Database per altri motori di ricerca, come Bing e Shodan, sono disponibili in risorse come il Google Hacking Diggity Project.

## Remediation

Analizza attentamente la sensibilità delle informazioni di design e di configurazione prima che vengano postate online.

Rivedi periodicamente la sensibiltà delle informazioni di design e di configurazione che sono postate online.