# WSTG-INFO-06

# Identify Application Entry Points

## Summary

L'enumerazione dell'app e della sua superficie d'attacco è un prerequisito chiave prima che qualsiasi test possa essere eseguito, dato che consente al tester di identificare le aree potenzialmente vulnerabili.
Questa sezione mira a identificare e mappare le aree dell'app su cui bisogna investigare dopo che l'enumerazione e il mapping sono stati completati.

## Test Objectives

Capire come sono formulate le richieste e le risposte scambiate con l'app.

## How to Test

Prima di qualsiasi test, il tester dovrebbe sempre acquisire una buona conoscenza dell'app e di come l'utente e i browser comunicano con essa.
Man mano che usa l'app, dovrebbe far attenzione a tutte le richieste HTTP, a ogni parametro e campo dei form che viene passato all'app.
Dovrebbe prestare attenzione a quando le richieste GET sono usate e a quando le richieste POST sono usate per passare dei parametri all'app.
Inoltre, dovrebbe far attenzione a quando gli altri metodi HTTP vengono usati su servizi RESTful.

Nota che per vedere i parametri inviati nel body di richieste come quelle POST, il tester potrebbe usare un tool come un intercepting proxy.
Analizzando la richiesta POST, il tester dovrebbe prendere nota dei campi nascosti dei form che vengono passati all'app, dato che potrebbero contenere informazioni sensibili, come informazioni sullo stato, quantità di oggetti, prezzo degli oggetti, che lo sviluppatore non vuole che l'utente veda o modifichi.

È molto utile usare un intercepting proxy e un foglio di calcolo per questa fase del testing.
Il proxy terrà traccia di ogni richiesta e risposta tra il tester e l'app.
In più, a questo punto, i tester di solito catturano ogni richiesta e risposta in modo da vedere esattamente ogni header, parametro, etc. che è stato passato all'app e cosa è stato restituito.
Questo processo potrebbe essere noioso, specialmente per siti interattivi estesi (es. app di una banca).
Tuttavia, l'esperienza insegnerà a guardare nel posto giusto e a velocizzare questo processo.

Man mano che il tester usa l'app, dovrebbe prendere nota di qualsiasi parametro URL interessante, header custom, o body delle richieste/risposte, e salvarli nel foglio di calcolo.
Il foglio di calcolo dovrebbe includere 
la pagina richiesta (potrebbe essere utile aggiungere il numero di richiesta del proxy, per eventuali riferimenti futuri), 
il parametro interessante, 
il metodo della richiesta, 
se l'accesso è autenticato o meno, 
se viene usato SSL, 
se fa parte di un processo multi-step, 
e altre note rilevanti.
Quando ha mappato ogni area dell'app, può procedere a testare le aree che ha identificato e prendere nota di ciò che funziona e di ciò che non funziona.
Il resto di questa guida identificherà come testare ognuna di queste aree di interesse, ma questa sezione deve essere rispettata prima che il vero e proprio testing abbia inizio.

Seguono alcuni punti di interesse per tutte le richieste e risposte.
La seguente sezione Requests si focalizza sui metodi GET e POST, dato che sono quelli maggiormente usati.
Nota che anche altri metodi, come PUT e DELETE, possono essere usati.
Spesso, queste richieste più rare, se consentite, possono presentare delle vulnerabilità.
In questa guida è presente una sezione speciale dedicata al testing di questi metodi HTTP.

### Requests

- identifica dove vengono usate le richieste GET e POST
- identifica tutti i parametri usati in una richiesta POST (nel body)
- nella richiesta POST, fai attenzione a qualsiasi parametro nascosto.
Quando viene inviata una POST tutti i campi del form (inclusi quelli nascosti) sono inviati nel body del messaggio HTTP.
Di solito non vengono visti a meno di usare un proxy o di analizzare il codice sorgente HTML.
Inoltre, la risposta ottenuta dall'app potrebbe essere diversa in base ai valori dei parametri nascosti
- identifica tutti i parametri usati nella richiesta GET, in particolare la query string
- identifica tutti i parametri della query string.
Di solito sono in coppia, nel formato `foo=bar`.
Nota anche che più parametri possono essere nella stessa query string separati da `&`, `\~`, `:`, o qualsiasi altro carattere speciale o encoding
- il tester deve identificare tutti i parametri (anche se encoded o cifrati) e capire quali sono elaborati dall'app.
Le sezioni successive della guida indicheranno come identificare questi parametri.
A questo punto, assicurati solo che ognuno di essi sia identificato.
- fai attenzione anche a qualsiasi header aggiuntivo o custom non molto diffuso (es. debug=False)

### Responses

- identifica dove vengono impostati, modificati o aggiunti i nuovi cookie
- identifica dove ci sono redirect, 400 status code, in particolare 403 Forbidden, e 500 internal server error durante l'uso normale dell'app
- nota anche dove viene usato un qualsiasi header interessante.
Per esempio, `Server: BIG-IP` indica che l'app si trova dietro un load balancer.
In questo caso, se un server è mal configurato, il tester potrebbe dover fare più richieste per accedere al server vulnerabile, in base al load balancing usato.

### Black-Box Testing

#### Testing for Application Entry Points

Seguono due esempi su come controllare gli entry point dell'app.

##### EXAMPLE 1

Questo esempio mostra una richiesta GET che acquisterebbe un oggetto da un app di shopping online.

```sh
GET /shoppingApp/buyme.asp?CUSTOMERID=100&ITEM=z101a&PRICE=62.50&IP=x.x.x.x HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=Z29vZCBqb2IgcGFkYXdhIG15IHVzZXJuYW1lIGlzIGZvbyBhbmQgcGFzc3dvcmQgaXMgYmFy
```

> Qui il tester dovrebbe annotare tutti i parametri della richiesta come CUSTOMERID, ITEM, PRICE, IP e il Cookie (che potrebbero essere i parametri encoded o usato per mantenere lo stato della sessione)

##### EXAMPLE 2

Questo esempio mostra una richiesta POST che dovebbe farti accedere all'app.

```sh
POST /KevinNotSoGoodApp/authenticate.asp?service=login HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=dGhpcyBpcyBhIGJhZCBhcHAgdGhhdCBzZXRzIHByZWRpY3RhYmxlIGNvb2tpZXMgYW5kIG1pbmUgaXMgMT
IzNA==
CustomCookie=00my00trusted00ip00is00x.x.x.x00

user=admin&pass=pass123&debug=true&fromtrustIP=true
```

> In questo esempio il tester dovrebbe annotare tutti i parametri come fatto prima, comunqe la maggior parte dei parametri sono passati nel body della richiesta e non nell'URL.
> Inoltre, nota che viene usato un header HTTP custom (`CustomCookie`).

### Gray-Box Testing

Il testing di entry point dell'app tramite la metodologia gray-box consisterebbe in quanto già descritto prima con una piccola aggiunta.
Nei casi in cui ci siano risorse esterne da cui l'app riceve dati e li elabora (come SNMP trap, messaggi di syslog, messaggi SMTP o SOAP da un altro server) un incontro con gli sviluppatori dell'app potrebbe aiutare a identificare le funzioni che accettano o si aspettano input utente e come è formattato.
Per esempio, lo sviluppatore potrebbe aiutare a capire come formulare una richiesta SOAP corretta che l'app accetta e dove risiede il servizio web (se il servizio web o qualsiasi altra funzione non è già stata identificata nel black-box testing).

#### OWASP Attack Surface Detector

Il tool Attack Surface Detector (ASD) analizza il codice sorgente e individua 
gli endpoint di una web app, 
i parametri accettati da questi endpoint, e 
il tipo di dato di questi parametri.
Questi includono gli endpoint che uno spider non sarebbe in grado di inviduare, o parametri opzionali mai usati lato client.
È inoltre in grado di tenere traccia dei cambiamenti della superficie d'attacco tra due verisoni dell'app.

L'ASD è disponibile come plugin sia per ZAP che per Burp Suite, e come tool Command Line Interface (CLI).
Il tool CLI esporta la superficie d'attacco in JSON, che può poi essere importato nel plugin di ZAP o Burp Suite.
È utile quando il codice sorgente non viene fornito direttamente al tester.
Per esempio, il tester può ottenere il json dal cliente che non vuole cedere il codice sorgente.

##### How to Use

Il file jar CLI può essere scaricato da [https://github.com/secdec/attack-surface-detector-cli/releases](https://github.com/secdec/attack-surface-detector-cli/releases).

Puoi eseguire il seguente comando per identificare gli endpoint a partire dal codice sorgente della web app.

`java -jar attack-surface-detector-cli-1.3.5.jar <source-code-path> [flags]`

Ecco un esempio del comando usato su OWASP RailsGoat.

```
$ java -jar attack-surface-detector-cli-1.3.5.jar railsgoat/
Beginning endpoint detection for '<...>/railsgoat' with 1 framework types
Using framework=RAILS
[0] GET: /login (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING};
FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
[1] GET: /logout (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[2] POST: /forgot_password (0 variants): PARAMETERS={email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '29'-'38')
[3] GET: /password_resets (0 variants): PARAMETERS={token=name=token, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '19'-'27')
[4] POST: /password_resets (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user=name=user, paramType=QUERY_STRING, dataType=STRING, confirm_password=name=confirm_password, paramType=QUERY_STRING, dataType=STRING};
FILE=/app/controllers/password_resets_controller.rb (lines '5'-'17')
[5] GET: /sessions/new (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
[6] POST: /sessions (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user_id=name=user_id, paramType=SESSION, dataType=STRING, remember_me=name=remember_me, paramType=QUERY_STRING, dataType=STRING, url=name=url, paramType=QUERY_STRING, dataType=STRING, email=name=email, paramType=QUERY_STRING, dataType=STRING};
FILE=/app/controllers/sessions_controller.rb (lines '11'-'31')
[7] DELETE: /sessions/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[8] GET: /users (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '9'-'11')
[9] GET: /users/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '13'-'15')
... snipped ...
[38] GET: /api/v1/mobile/{id} (0 variants): PARAMETERS={id=name=id, paramType=QUERY_STRING, dataType=STRING, class=name=class, paramType=QUERY_STRING, dataType=STRING};
FILE=/app/controllers/api/v1/mobile_controller.rb (lines '8'-'13')
[39] GET: / (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING};
FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')

Generated 40 distinct endpoints with 0 variants for a total of 40 endpoints
Successfully validated serialization for these endpoints
0 endpoints were missing code start line
0 endpoints were missing code end line
0 endpoints had the same code start and end line
Generated 36 distinct parameters
Generated 36 total parameters
- 36/36 have their data type
- 0/36 have a list of accepted values
- 36/36 have their parameter type
--- QUERY_STRING: 35
--- SESSION: 1
Finished endpoint detection for '<...>/railsgoat'
----------
-- DONE --
0 projects had duplicate endpoints
Generated 40 distinct endpoints
Generated 40 total endpoints
Generated 36 distinct parameters
Generated 36 total parameters
1/1 projects had endpoints generated
To enable logging include the -debug argument
```

Puoi anche generare il json usando il flag `-json`, che può essere usato dal plugin di ZAP o Burp Suite.

## Tools

- OWASP Zed Attack Proxy (ZAP)
- Burp Suite
- Fiddler