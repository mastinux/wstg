# WSTG-ATHN-04

# Testing for Bypassing Authentication Schema

## Summary

In sicurezza informatica, l'autenticazione è il processo finalizzato alla verifica dell'identità digitale del mittente di una comunicazione.
Un esempio è il processo di login.
Testare lo schema di autenticazione significa capire come funziona il processo di autenticazione e usare queste informazioni per raggirarlo.

Anche se la maggior parte delle app richiede l'autenticazione per ottenere l'accesso alle informazioni private o per eseguire dei task, non tutti i metodi di autenticazione sono in grado di fornire un'adeguata sicurezza.
Negligenza o ignoranza delle minacce di sicurezza spesso sono causa di schema di autenticazione che possono essere raggirati semplicemente evitando la pagina di login e invocando direttamente una pagina interna che era stata pensata per essere acceduta solo dopo l'autenticazione.

Inoltre, spesso è possibile raggirare le misure di autenticazione alterando le richieste e facendo credere all'app che l'utente sia già autenticato.
Si può fare ciò modificando i parametri dell'URL, alterando il form, o manipolando le sessioni.

I problemi relativi agli schema di autenticazione possono essere individuati in diverse fasi del software development life cycle (SDLC), come le fasi di design, development, e deployment.

- nella fase di design 
gli errori possono includere 
una definizione errata delle sezioni dell'app da proteggere,
la scelta di protocolli di cifratura non robusti per la protezione della trasmissione di credenziali,
e molti altri
- nella fase di development
gli errori possono includere
l'implementazione errata delle funzionalità di input validation
o il mancato rispetto delle best practice per un dato linguaggio
- nella fase di deployment,
ci potrebbero essere problemi durante il setup dell'app (attività di installazione e di configurazione) a causa di lacune tecniche o di mancanza di documentazione

## How to Test

### Black-Box Testing

Ci sono diversi metodi per raggirare lo schema di autenticazione usato da un web app:

- direct page request
- parameter modification
- session ID prediction
- SQL injection

#### Direct Page Request

Se una web app implementa il controllo accessi solo sulla pagina di login, lo schema di autenticazione potrebbe essere raggirato.
Per esempio, se un utente richiede direttamente una pagina diversa tramite forced browsing, quella pagina potrebbe non verificare le credenziali dell'utente prima di dare l'accesso.
Prova ad accedere direttamente a una pagina protetta tramite l'indirizzo nella barra degli indirizzi del tuo browser per testare questo metodo.

#### Parameter Modification

Un altro problema relativo al design dei meccanismi di autenticazione si ha quando l'app verifica un login avvenuto con successo sulla base di parametri con valore fisso.
Un utente potrebbe modificare questi parametri per ottenere l'accesso ad aree protette senza dover fornire credenziali valide.
Nell'esempio sottostante, il parametro "authenticated" viene cambiato con il valore "yes", consentendo all'utente di ottenere l'accesso.
In questo esempio, il parametro è nell'URL, ma si può usare un proxy per modificarlo, specialmente quando i parametri sono inviati come elementi di una richiesta POST o quando sono memorizzati in un cookie.

```sh
http://www.site.com/page.asp?authenticated=no

raven@blackbox /home $nc www.site.com 80
GET /page.asp?authenticated=yes HTTP/1.0


HTTP/1.1 200 OK
Date: Sat, 11 Nov 2006 10:22:44 GMT
Server: Apache
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>You Are Authenticated</H1>
</BODY></HTML>
```

#### Session ID Prediction

Molte web app gestiscono l'autenticazione usando i session identifier (session IDs).
Quindi, se la gerazione di un session ID è predicibile, un utente malevolo potrebbe essere in grado di trovare un session ID valido e ottenere un accesso non autorizzato all'app, impersonando un utente precedentemente autenticato.

#### SQL Injection

L'SQL injection è una tecnica di attacco ben consociuta.
Questa sezione non spiega questa tecnica nel dettaglio dato che ci sono molte altre sezioni in questa guida che spiegano le tecniche di injection.

Prova a usare le tecniche di SQL injection per raggirare il form di autenticazione.

### Grey-Box Testing

Se un attaccante è stato in grado di recuperare il codice sorgente dell'app sfruttando un'altra vulnerabilità (es. directory traversal), o da un repository web (app open source), può essere possibile preparare un attacco ad hoc contro l'implementazione del processo di autenticazione.

Nel seguente esempio (PHPBB 2.0.13 - Authentication Bypass Vulnerability), alla riga 5 la funzione `unserialize()` fa il parsing di un cookie e imposta i valori nell'array `$row`.
Alla riga 10 l'hash MD5 della password dell'utente memorizzato nel database di backend viene confrontato con il valore ricevuto.

```
1.  if ( isset($HTTP_COOKIE_VARS[$cookiename . '_sid']) ||
2.  {
3.  $sessiondata = isset( $HTTP_COOKIE_VARS[$cookiename . '_data'] ) ?
4. 
5.  unserialize(stripslashes($HTTP_COOKIE_VARS[$cookiename . '_data'])) : array();
6. 
7.  $sessionmethod = SESSION_METHOD_COOKIE;
8.  }
9. 
10. if( md5($password) == $row['user_password'] && $row['user_active'] )
11. 
12. {
13. $autologin = ( isset($HTTP_POST_VARS['autologin']) ) ? TRUE : 0;
14. }
```

In PHP, un confronto tra una string e un boolean (1 - "TRUE") è sempre "TRUE", quindi inviando la seguente string (la arte importante è `"b:1"`) alla funzione `unserialize()`, è possibile raggirare il controllo di autenticazione:

`a:2:{s:11: “ autologinid ” ;b:1;s:6: “ userid ” ;s:1: “ 2 ” ;}`

## Tools

- WebGoat
- OWASP Zed Attack Proxy