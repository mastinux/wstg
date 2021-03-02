# WSTG-ATHZ-03

# Testing for Privilege Escalation

## Summary

Questa sezione descrive il problema della privilege escalation da un livello a un altro.
Durante questa fase, il tester dovrebbe verificare che per un utente non sia possibile modificare i propri privilegi o ruoli nell'app in modo da realizzare attacchi di privilege escalation.

La privilege escalation si verifica quando un utente ottiene l'accesso a più risorse o più funzionalità rispetto a quelle che gli sono normalemnte consentite, 
e tali cambiamenti dovrebbero essere impediti dall'app.
Ciò è causato da una vulnerabilità nell'app.
Il risultato è che l'app esegue azioni con privilegi più elevati rispetto a quelli pensati originariamente dallo sviluppatore o dall'amministratore di sistema.

Il grado di escalation dipende da quali privilegi l'attaccante è autorizzato a possedere, e quali privilegi possono essere ottenuti a seguito di un exploit.
Per esempio, un errore di programmazione che consente a un utente di ottenere privilegi extra dopo l'autenticazione limita il grado di escalation, dato che l'utente è già autorizzato ad avere alcuni privilegi.
Allo stesso modo, un attaccante remoto che ottiene privilegi da superuser senza alcuna autenticazione presenta un grado di escalation più grande.

Di solito, ci si riferisce 
a vertical escalation quando è possibile accedere a risorse concesse a utenti con maggiori privilegi (es. acquisire privilegi amministrativi per l'app), e 
a horizontal escalation quando è possibile accedere a risorse concesse ad account con stessi privilegi (es. in un'app di online banking, l'accesso a informazioni relative a utenti diversi).

## How to Test

### Testing for Role/Privilege Manipulation

In ogni porzione dell'app in cui l'utente può creare informazioni nel database (es. fare un pagamento, aggiungere un contatto, o inviare un messaggio), può ricevere informazioni (estratto conto, dettagli dell'ordine, etc.), o può eliminare informazioni (cancellare utenti, messaggi, etc.), è necessario registrare tale funzionalità.
Il tester dovrebbe provare ad accedere a tali funzioni con un utente diverso per verificare se è possibile accedere a funzioni che non dovrebbero essere permesse in base al ruolo/privilegio dell'utente (ma potrebbero essere concessi con un altro utente).

#### Manipulation of User Group

Per esempio: la seguente HTTP POST consente all'utente che appartiene a `grp001` di accedere all'ordine #0001:

```
POST /user/viewOrder.jsp HTTP/1.1
Host: www.example.com
...

groupID=grp001&orderID=0001
```

Verifica se un utente che non appartiene a `grp001` può modificare il valore del parametro `groupId` e `orderId` per ottenere l'accesso a quei dati privilegiati.

#### Manipulation of User Profile

Per esempio: la seguente risposta del server monstra un campo nascosto nell'HTML restituito all'utente dopo l'autenticazione.

```
HTTP/1.1 200 OK
Server: Netscape-Enterprise/6.0
Date: Wed, 1 Apr 2006 13:51:20 GMT
Set-Cookie: USER=aW78ryrGrTWs4MnOd32Fs51yDqp; path=/; domain=www.example.com
Set-Cookie: SESSION=k+KmKeHXTgDi1J5fT7Zz; path=/; domain= www.example.com
Cache-Control: no-cache
Pragma: No-cache
Content-length: 247
Content-Type: text/html
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Connection: close

<form name="autoriz" method="POST" action = "visual.jsp">
<input type="hidden" name="profile" value="SysAdmin">\
<body onload="document.forms.autoriz.submit()">
</td>
</tr>
```

Cosa succede se il tester modifica il valore della variabile `profile` in `SysAdmin`?
È possibile diventare administrator?

#### Manipulation of Condition Value

Per esempio: il server invia un messaggio di errore contenuto come valore nel parametro specifico in un insieme di codici di risposta, come segue:

```
@0`1`3`3``0`UC`1`Status`OK`SEC`5`1`0`ResultSet`0`PVValid`-1`0`0` Notifications`0`0`3`Command
Manager`0`0`0` StateToolsBar`0`0`0`
StateExecToolBar`0`0`0`FlagsToolBar`0
```

Il server dà una fiducia implicita all'utente.
Crede che l'utente risponderà con il messaggio precedente chiudendo la sessione.

In questa condizione, verifica che non sia possibile fare privilege escalation modificando i valori del parametro.
In questo particolare esempio, modifica il valore di `PVValid` da `-1` a `0`, 
tramite cui potrebbe essere possibile autenticarsi come amministratore sul server.

#### Manipulation of IP Address

Alcuni siti web limitano l'accesso o contano il numero di tentativi di login falliti in base all'indirizzo IP.

Per esempio:

`X-Forwarded-For: 8.1.1.1`

In questo caso, se il sito valuta il valore dell'header `X-Forwarded-For` come indirizzo IP del client, 
il tester potrebbe cambiare il valore dell'indirizzo IP nell'header `X-Forwarded-For` per raggirare l'identificazione tramite indirizzo IP sorgente.

### URL Traversal

Prova a navigare il sito web e controlla se alcune pagine possono avere lacune sui controlli di autorizzazione.

Per esempio:

`/../.././userInfo.html`

## WhiteBox

Se i controlli di autorizzazione sono eseguiti solo in base a una parte dell'URL, allora è probabile che i tester o gli hacker possano raggirare i controlli tramite l'encoding dell'URL.

Per esempio: `startsWith()`, `endswith()`, `contains()`, `indexOf()`.

### Weak SessionID

I Session ID deboli potrebbero essere vulnerabili ad attacchi di bruteforce.
Per esempio, un sito usa MD5(password + userID) come sessionID.
Allora, i tester potrebbero provare a generare il sessionID per gli altri utenti.

## Tools

OWASP ZAP