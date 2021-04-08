# WSTG-SESS-02

# Testing for Cookies Attributes

## Summary

I Web Cookie sono spesso un attack vector chiave per utenti malevoli (che di solito hanno come obiettivo ad altri utenti). 
L'app dovrebbe sempre adottare le adeguate protezioni rispetto ai cookie.

HTTP è un protocollo stateless, nel senso che non tiene traccia delle richieste inviate da uno stesso utente.
Per risolvere questo problema, le sessioni vengono create e agganciate alle richieste HTTP.
I browser, come discusso in [Testing Browser Storage](./WSTG-CLNT-12.md), contengono una moltitudine di meccanismi di storage.
In quella sezione della guida, ognuno di essi viene presentato accuratamente.

Il meccanismo di storage più usato nei browser è quello basato sul cookie.
I cookie possono essere impostati dal server, includendo un header `Set-Cookie` nella risposta HTTP o tramite JavaScript.
I cookie possono essere usati per una moltitudine di scopi, come:

- gestione della sessione
- personalizzazione
- tracciamento

Per proteggere i dati dei cookie, l'industry ha sviluppato dei meccanismi per confinare i cookie e limitarne la superficie d'attacco.
Nel corso del tempo i cookie sono divenuti il meccanismo di storage preferito per le web app, dato che forniscono una grande flessibilità di utilizzo e protezione.

I modi per proteggere i cookie sono:

- attributi
- prefissi

## Test Objectives

I tester dovrebbero essere a conoscenza di tutti i cookie che vengono impostati, e che siano state implementate adeguate configurazioni di sicurezza.

## How to Test

Il tester dovrebbe verificare che gli attributi e i prefissi dei cookie siano stati usati adeguatamente dall'app.
I cookie possono essere controllati usando un proxy, o ispezinando la cookie jar del browser.

### Cookie Attributes

#### Secure Attribute

L'attributo `Secure` indica al browser di inviare il cookie solo se la richiesta è trasmessa su un canale sicuro come `HTTPS`.
Ciò impedisce che il cookie venga trasmesso su richieste non cifrate.
Se l'app può essere acceduta sia su HTTP che su HTTPS, un attaccante potrebbe essere in grado di forzare l'utente a inviare i cookie su richieste non protette.

#### HttpOnly Attribute

L'attributo `HttpOnly` viene usato per impedire attacchi come session leakage, dato che evita che il cookie venga acceduto da script client side come JavaScript.

> Ciò non limita l'intera superficie d'attacco dei XSS, dato che un attaccante potrebbe inviare ugualmente una richiesta per conto dell'utente, ma è un enorme limite agli attack vector dei XSS

#### Domain Attribute

L'attributo `Domain` viene usato per confrontare il dominio del cookie con il dominio del server verso cui la richiesta viene inviata.
Se il dominio coincide oppure è un suo sottodominio, allora viene controllato l'attributo `path`.

Nota che solo gli host che appartengono allo specifico dominio possono impostare un cookie per quel dominio.
Inoltre, l'attributo `domain` non può essere un top level domain (come `.gov` o `.com`) in modo da impedire ai server di impostare cookie arbitrari per un altro dominio (come impostare un cookie per `owasp.org`).
Se l'attributo domain non è impostato, allora l'host name del server che ha generato il cookie viene usato come valore di default per `domain`.

Per esempio, se un cookie viene impostato da un'app su 
`app.mydomain.com` senza l'attributo domain impostato, il cookie dovrebbe essere inserito nelle richieste successive ad 
`app.mydomain.com` e i suoi sottodomini (come 
`hacker.app.mydomain.com`), ma non per 
`otherapp.mydomain.com`.
Se uno sviluppatore vuole rilassare questa restrizione, può impostare l'attributo `domain` a 
`mydomain.com`.
In questo caso il cookie verrebbe inviato in tutte le richieste ad 
`app.mydomain.com` e ai sottodomini di 
`mydomain.com`, come 
`hacker.app.mydomain.com`, e anche 
`bank.mydomain.com`.
Se ci fosse un server vulnerabile in un sottodominio (per esempio, 
`otherapp.mydomain.com`) e l'attributo `domain` fosse stato impostato in modo troppo rilassato (per esempio, 
`mydomain.com`), allora il server vulnerabile potrebbe essere usato per recuperare i cookie (come i token di sessione) su tutto lo scope di `mydomain.com`.

#### Path Attribute

190

#### Expires Attribute

#### SameSite Attribute

##### Strict Value

##### Lax Value

##### None Value

### Cookies Prefixes
