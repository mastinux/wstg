# WSTG-ATHZ-02

# Testing for Bypassing Authorization Schema

## Summary

Questo test ha l'obiettivo di verificare come lo schema autorizzativo è stato implementato per ogni ruolo o privilegio per ottenere l'accesso a funzioni o risorse riservate.

Per ogni ruolo che il tester possiede durante i test, e per ogni funzione e richiesta che l'app esegue durante la fase successiva all'autenticazione, è necessario verificare:

- è possibile accedere alla risorsa anche se l'utente non è autenticato?
- è possibile accedere alla risorsa dopo il log-out?
- è possibile accedere a funzioni e risorse che dovrebbero essere accessibili a utenti che hanno ruoli o privilegi diversi?

Prova ad accedere all'app come utente amministrativo e tieni traccia di tutte le funzioni amministrative.

- è possibile accedere alle funzioni amministrative anche se il tester è loggato come utnete con privilegi standard?
- è possibile usare queste funzioni amministrative come utente con un ruolo diverso e per il quale quell'azione dovrebbe essere negata?

## How to Test

### Testing for Access to Administrative Functions

Per esempio, supponi che la funzione `addUser.jsp` sia parte del menu amministrativo dell'app, e che sia accessibile richiedendo la seguente URL:

`https://www.example.com/admin/addUser.jsp`

Poi, che la seguente richiesta HTTP venga generata quando viene chiamata la funzione:

```
POST /admin/addUser.jsp HTTP/1.1
Host: www.example.com
[other HTTP headers]

userID=fakeuser&role=3&group=grp001
```

Cosa succede se un utente non amministrativo prova a eseguire questa richiesta?
L'utente viene creato?
In caso positivo, il nuovo utente creato può usare i suoi privilegi?

### Testing for Access to Resources Assigned to a Different Role

Per esempio un'app usa una directory condivisa per memorizzare file PDF temporanei per utenti diversi.
Supponi che `documentABC.pdf` possa essere acceduto solo dall'utente `test1` con `roleA`.
Devi verificare se l'utente `test2` con `roleB` può accedere alla risorsa.

### Testing for Special Request Header Handling

Alcune app supportano header non standard come `X-Original-URL` o `X-Rewrite-URL` per consentire l'override dell'URL nelle richieste con quella specificata nel valore dell'header.

Questo comportamento può essere sfruttato in situazioni in cui l'app è dietro un componente che applica restrizioni sul controllo accessi in base all'URL della richiesta.

Il tipo di restrizioni sul controllo accessi basato sull'URL della richiesta può essere, per esempio, bloccare l'accesso da Internet alla console amministrativa esposta su `/console` o `/admin`.

Per individuare il supporto degli header `X-Original-URL` o `X-Rewrite-URL`, puoi seguire i seguenti passi.

1. invia una richiesta normale senza alcun header X-Original-URL o X-Rewrite-URL

```
GET / HTTP/1.1
Host: www.example.com
[other standard HTTP headers]
```

2. invia una richiesta con un header X-Original-URL che punta a una risorsa non esistente

```
GET / HTTP/1.1
Host: www.example.com
X-Original-URL: /donotexist1
[other standard HTTP headers]
```

3. invia una richiesta con un header X-Rewrite-URL che punta a una risorsa non esistente

```
GET / HTTP/1.1
Host: www.example.com
X-Rewrite-URL: /donotexist2
[other standard HTTP headers]
```

Se la risposta ad almeno una delle ultime due richieste contiene indicazioni sul fatto che la risorsa non è stata trovata, ciò indica che l'app supporta questi header.
Questi indicatori potrebbero essere lo status 404, o un messaggio "resource not found" nel response body.

Una volta che il supporto dell'header `X-Original-URL` o `X-Rewrite-URL` è stato validato, 
il tentativo di raggirare le restrizioni di access control può essere condotto inviando la particolare richiesta ma 
specificando un'URL "consentita" dal componente di front-end come URL della richiesta principale e 
specificando l'URL obiettivo reale nell'header `X-Original-URL` o `X-Rewrite-URL` in base a quello supportato.
Se entrambi gli header sono supportati, verificane uno dopo l'altro per vedere quale consente il bypass delle restrizioni.

## Tools

- OWASP ZAP