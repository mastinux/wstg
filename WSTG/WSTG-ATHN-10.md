# WSTG-ATHN-10

# Testing for Weaker Authentication in Alternative Channel

## Summary

Anche se i meccanismi di autenticazione primaria non includono alcuna vulnerabilità, potrebbero essere presenti delle vunlerabilità negli user channel di autenticazione alternativi legittimi.
Dovrebbero essere eseguiti dei test per identificare i channel alternativi e, in base allo scope dei test, identificare le vulnerabilità.

Gli user channel alternativi potrebbero essere usati per raggirare il channel primario, o potrebbero esporre informazioni che possono essere usate per aiutare un utente malevolo ad eseguire un attacco contro il channel primario.
Alcuni di questi channel potrebbero essere web app separate che usano host name e path diversi.
Per esempio:

- siti web standard
- siti web ottimizzati per mobile o per specifici device
- siti ottimizzati per l'accessibilità
- siti alternativi per paese e lingua
- siti paralleli che usano lo stesso account utente (es. 
un altro sito web che offre funzionalità diverse della stessa organizzazione, 
un sito partner con cui vengono condivisi gli account utente)
- versioni di development, test, UAT e staging del sito web standard

Ma potrebbero essere anche altri tipi di app o processi di business:

- app mobile
- app desktop
- operatori di call center
- risposta con voce interattiva o sistema phone tree

Nota che il focus di questo test è sui channel alternativi;
alcune alternative per l'autenticazione potrebbero apparire come diverso contenuto fornito dallo stesso sito web e sarà sicuramente nello scope del test.
Ciò non viene discusso qui, e dovrebbe essere stato identificato durante l'information gathering e i primi test sull'autenticazione.
Per esempio:

- progressivo arricchimento e lieve degradazione che cambia la funzionalità
- uso del sito senza cookie
- uso del sito senza JavaScript
- uso del sito senza plugin come Flash o Java

Anche se lo scope del test non permette di verificare i channel alternativi, la loro esistenza dovrebbe essere documentata.
Ciò potrebbe minare la sicurezza dei meccanismi di autenticazione e potrebbe essere un precodizione per lo svolgimento di altri test.

## Example

Il sito primario è:

`http://www.example.com`

e le funzioni di autenticazione hanno sempre luogo in pagine che usano TLS:

`https://www.example.com`

Tuttavia, esiste un sito ottimizzato per mobile che non usa TLS e ha un meccanismo di recovery della password più debole:

`http://m.example.com/myaccount`

## How to Test

### Understand the Primary Mechanism

Testa completamente le funzioni di autenticazione primaria del sito web.
Ciò dovrebbe identificare come vengono creati, attivati o cambiati gli account e come le password sono recuperate, reimpostate o cambiate.
Bisognerebbe acquisire conoscenza di qualsiasi misura di protezione per l'autenticazione e l'autorizzazione con privilegi elevati.
Questi prerequisiti sono necessari per poter confrontarli con i channel alternativi.

### Identify Other Channels

Gli altri channel possono essere individuati usando i seguenti metodi:

- leggendo il cotenuto del sito, specialmente l'home page, il contact us, le help page, gli articoli di supporto e le FAQ, i Terms & Conditions, le privacy notices, il file robots.txt e qualsiasi file sitemap.xml
- cercando nei log del proxy HTTP, creato durante l'information gathering e il testing, string come 
"mobile",
"android",
"blackberry",
"ipad",
"iphone",
"mobile app",
"e-reader",
"wireless",
"auth",
"sso",
"single sign on"
nei path delle URL e nel body

Per ogni possibile channel verifica se gli account utente sono condivisi tra di essi, o hanno accesso alla stessa funzionalità.

### Enumerate Authentication Functionality

Per ogni channel alternativo in cui gli account utente o le funzionalità sono condivise, identifica se tutte le funzioni di autenticazione del channel primario sono disponibili, e se esiste qualcosa in più.
Potrebbe essere utile creare una griglia come la seguente

|Primary|Mobile|Call Center|Partner Website|
|-|-|-|-|
|Register|Yes|-|-|
|Log in|Yes|Yes|Yes(SSO)|
|Log out|-|-|-|
|Password Reste|Yes|Yes|-|
|Change Password|Yes|-|-|

In questo esempio, il mobile ha una funzione extra di "change password" ma non offre quella di "log out".
Sono disponibili alcune funzionalità anche tramite la chiamata al call center.
Il call center potrebbe essere interessante, dato che i loro controlli di conferma di identità potrebbero essere più deboli di quelli del sito web, facendo sì che questo channel possa essere usato per eseguire un attacco contro l'account dell'utente.

Durante l'enumerazione dei channel è meglio prendere nota di come viene gestita la sessione, per verificare se la sessione si sovrappone tra channel diversi (es. 
cookie con scope uguale al domain name padre,
sessioni concorrenti consentite tra i channel ma non sullo stesso channel)

### Review and Test

I channel alternativi dovrebbero essere menzionati nel testing report, anche se vengono segnati come "information only" o "out of scope".
In alcuni casi lo scope del test potrebbe includere i channel alternativi (es. perchè è un altro path verso l'host name obiettivo), o potrebbero essere aggiunti allo scope dopo una discussione con i possessori di tutti i channel.
Se il testing è consentito e autorizzato, 
dovresti eseguire tutti gli altri test presenti in questa guida, e 
confrontarli con il channel primario.

## Related Test Cases

Bisognerebbe usare i test case per tutti gli altri test sull'autenticazione.

## Remediation

Assicurati che venga applicata un'autentication policy consistente su tutti i channel in modo che siano tutti ugualmente sicuri.