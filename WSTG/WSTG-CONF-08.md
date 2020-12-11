# WSTG-CONF-08

# Test RIA Cross Domain Policy

## Summary

Le Rich Internet Application (RIA) hanno adottato i file di policy crossdomain.xml di Adobe per consentire l'accesso controllato cross domain per l'utilizzo di dati e servizi tramite l'uso di tecnologie come Oracle Java, Silverlight, e Adobe Flash.
In questo modo, un dominio può concedere l'accesso remoto ai suoi servizi a un dominio diverso.
Tuttavia, spesso i file di policy che descrivono le restrizioni di accesso sono mal configurati.
Le mal configurazioni dei file di policy favoriscono attacchi di Cross-Site Request Forgery, e possono consentire a terze parti l'accesso a dati sensibili di un utente.

### What are cross-domain policy files?

Un file di cross-domain policy specifica i permessi che un web client come Java, Adobe Flash, Adobe Reader, etc. usa per accedere a dati ospitati da domini diversi.
Per Silverlight, Microsoft ha adottato un sottoinsieme di crossdomain.xml di Adobe, e ha creato in più il suo file di policy cross-domain: clientaccesspolicy.xml.

Quando un web client rileva che una risorsa deve essere richiesta a un altro dominio, 
prima cerca un file di policy nel diminio target per determinare se l'esecuzione delle richieste, inclusi gli header, e le connessioni socket-based sono consentiti.

I file master di policy si trovano nella root del dominio.
Un client potrebbe essere configurato in modo da caricare un file di policy diverso ma prima controlla sempre la presenza del file master di policy per assicurarsi che questo consenta il file custom di policy.

#### Crossdomain.xml vs Clientaccesspolicy.xml

La maggior parte delle RIA app supportano crossdomain.xml.
Tuttavia nel caso di Silverlight, funziona solo se crossdomain.xml specifica che l'accesso è consentito da qualsiasi dominio.
Per un controllo più granulare con Silverlight, bisogna usare clientaccesspolicy.xml.

I file di policy concedono diversi tipi di permessi:

- file di policy accettati (i file master di policy possono disabilitare o restringere specifici file di policy)
- permessi sui socket
- permessi sugli header
- permessi sull'accesso HTTP/HTTPS
- consentire l'accesso basato su credenziali crittografiche

Un esempio di policy eccessivamente permissiva è la seguente:

```xml
<?xml version="1.0"?>
<!DOCTYPE cross-domain-policy SYSTEM
"http://www.adobe.com/xml/dtds/cross-domain-policy.dtd">
<cross-domain-policy>
	<site-control permitted-cross-domain-policies="all"/>
	<allow-access-from domain="*" secure="false"/>
	<allow-http-request-headers-from domain="*" headers="*" secure="false"/>
</cross-domain-policy>
```

### How can cross domain policy files be abused?

- policy cross-domain eccessivamente permissiva
- generazione di risposte server che possono essere trattate come file di policy cross-domain
- uso di funzionalità di upload di file che possono essere usati come file di policy cross-domain

### Impact of Abusing Cross-Domain Access

- inefficacia delle protezioni CSRF
- lettura di dati limitati o diversamente protetti dalle policy cross-domain

## How to Test

### Testing for RIA Policy Files Weakness

Per testare le vulnerabilità dei file di policy RIA 
il tester dovrebbe recuperare il file di policy crossdomain.xml e clientaccesspolicy.xml dalla root dell'app e da ogni folder trovato.

Per esempio, se l'URL dell'app è `http://www.owasp.org`, il tester dovrebbe provare a scaricare i file `http://www.owasp.org/crossdomain.xml` e `http://www.owasp.org/clientaccesspolicy.xml`.

Dopo aver recuperato tutti i file di policy, bisognerebbe controllare i permessi secondo il principio del least privilege.
Le richieste dovrebbero provenire solo dai domini, dalle porte, o dai protocolli necessari.
Le policy eccessivamente permissive dovrebbero essere evitate.
Le policy contenenti `*` dovrebbero essere esaminate attentamente.

#### Example

```xml
<cross-domain-policy>
	<allow-access-from domain="*" />
</cross-domain-policy>
```

##### Result Expected

- una lista di file di policy trovati
- una lista di impostazioni deboli nelle policy

## Tools

- nikto
- OWASP ZAP
- W3af