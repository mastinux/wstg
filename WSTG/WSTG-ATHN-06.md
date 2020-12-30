# WSTG-ATHN-06

# Testing for Browser Cache Weaknesses

## Summary

In questa fase il tester controlla che l'app istruisca correttamente il browser per il mantenimento di dati sensibili.

I browser possono memorizzare informazioni per scopi di caching o di cronologia.
Il caching viene usato per migliorare le performance, in modo che le informazioni mostrate precedentemente non siano scaricate di nuovo.
I meccanismi della cronologia sono usati per comodità dell'utente, in modo che veda esattamente ciò che ha visto quando la risorsa è stata recuperata.
Se all'utente sono mostrate informazioni sensibili (come il suo indirizzo, i dettagli della carta di credito, il Social Security Number, o l'username), allora queste informazioni possono essere memorizzate per motivi di caching o di cronologia, e quindi sono recuperabili tramite l'analisi della cache del browser o semplicemente premendo il button Back del browser.

## Test Objectives

L'obiettivo di questo test è valutare se l'app memorizza o meno informazioni sensibili in posizioni accessibili al client o in un modo che non viene impedito l'accesso o verificare al di fuori di una sessione autenticata e autorizzata.
Specialmente, viene verificato se informazioni sensibili sono memorizzate:

- su disco o in memoria, dove potrebbero essere recuperate dopo il loro utilizzo normale
- in modo tale che usando il button Back potrebbe consentire all'utente (o all'attaccante) di tornare a uno screen precedentemente visualizzato

## How to Test

### Browser History

Tecnicamente, il button Back è relativo alla cronologia non è una cache (vedi https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.13).
La cache e la cronologia sono due diverse entità.
Comunque, condividono le stesse debolezze nel mostrare informazioni sensibili precedentemente mostrate.

Il primo e più semplice test consiste nell'inserire informazioni sensibili nell'app e fare log out.
Poi il tester clicca sul button Back del browser per verificare se le informazioni sensibili precedente mostrate sono ancora accessibili senza autenticazione.

Se il tester premendo sul button Back può accedere alle pagine precedenti ma non può accedere a quelle nuove, allora non è un problema di autenticazione, ma un problema di cronologia del browser.
Se le pagine contengono informazioni sensibili, significa che l'app non ha impedito al browser di memorizzarle.

L'autenticazione non deve essere necessariamente coinvolta nei test.
Per esempio, quando un utente inserisce la sua email per registrarsi a una newsletter, queste informazioni potrebbero essere recuperabili se non sono correttamente gestite.

Il back Button può essere bloccato quando mostra dati sensibili.
Si può:

- restituire la pagina su HTTPS
- impostare `Cache-Control: must-revalidate`

### Browser Cache

Qui il tester controlla che l'app non riveli informazioni sensibili nella cache del browser.
Per fare ciò, puoi usare un proxy (come OWASP ZAP) e cercare nelle risposte del server, controllando che per ogni pagina che contiene informazioni sensibili il server istruisca il browser in modo che non faccia caching dei dati.
Le direttive che possono essere usate negli header delle risposte HTTP sono:

- `Cache-Control: no-cache, no-store`
- `Expires: 0`
- `Pragma: no-cache`

Queste direttive sono di solito robuste, anche se potrebbero essere necessari flag aggiuntivi per l'header `Cache-Control` per meglio prevenire i file legati al file system.
Queste direttive sono:

- `Cache-Control: must-revalidate, max-age=0, s-maxage=0`

```
HTTP/1.1:
Cache-Control: no-cache
```

```
HTTP/1.0:
Pragma: no-cache
Expires: 0
```

Per esempio, se stai testando un'app di e-commerce, dovresti cercare in tutte le pagine che contengono un numero di carta di credito o altre informazioni finanziarie, e controllare che tutte queste pagine impongano la direttiva `no-cache`.
Se trovi delle pagine che contengono informazioni critiche ma che non istruiscono il browser per evitare che faccia il caching del loro contenuto, allora le informazioni sensibili verranno memorizzate su disco, e puoi fare una contro verifica semplicemente cercando la pagine nella cache del browser.

La posizione esatta in cui tali informazioni sono memorizzate dipende dal sistema operativo del client e dal browser usato.
Ecco alcuni esempi:

- Mozilla Firefox:
	- Unix/Linux: `~/.cache/mozilla/firefox/`
	- Windows: `C:\Users\<user_name>\AppData\Local\Mozilla\Firefox\Profiles\<profile-id>\Cache2\`
- Internet Explorer:
	- `C:\Users\<user_name>\AppData\Local\Microsoft\Windows\INetCache\`
- Chrome:
	- Windows: `C:\Users\<user_name>\AppData\Local\Google\Chrome\User Data\Default\Cache`
	- Unix/Linux: `~/.cache/google-chrome`

### Reviewing Cached Information

Firefox offre una funzionalità per vedere le informazioni in cache, che può essere usata a beneficio del tester.
Ovviamente sono state prodotte varie estensioni, e app esterne che potresti usare per Chrome, Internet Explorer o Edge.

I dettagli della cache sono anche disponibili tramite i developer tool in molti browser moderni, come Firefox, Chrome ed Edge.
Con Firefox è anche possibile usare l'URL `about:cache` per controllare i dettagli della cache.

### Check Handling for Mobile Browsers

La gestione delle direttive sulla cache possono essere completamente diverse sui browser mobile.
Quindi, dovresti avviare una nuova sessione con la cache vuota e usare le feature come il Device Mode di Chrome o il Responsive Design Mode di Firefox per testare i concetti spiegati prima.

Inoltre, i proxy come ZAP e Burp Suite permettono di specificare quale `User-Agent` far inviare dallo spider/crawler.
Potrebbe corrispondere alla string di un browser mobile ed essere usato per vedere quali direttive sul caching sono restituite dall'app.

## Gray-Box Testing

La metodologia per il gray-box testing è equivalente al black-box testing, dato che in entrambi gli scenari i tester hanno accesso completo agli header della risposta del server e al codice HTML.
Comunque, con il gray-box testing, il tester potrebbe aver accesso alle credenziali dell'account che gli permettono di testare pagine sensibili che sono accessibli solo agli utenti autenticati.

## Tools

- OWASP Zed Attack Proxy