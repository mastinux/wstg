# WSTG-IDNT-04

# Testing for Account Enumeration and Guessable User Account

## Summary

Lo scopo di questo test è verificare se è possibile collezionare un insieme di username interagendo con i meccanismi di autenticazione dell'app.
Questo test sarà utile per i test sul brute force, in cui il tester verifica se, dato un username valido, è possibile trovare la password corrispondente.

Spesso, le web app rivelano quando un utente esiste nel sistema, a causa di una mis-configuration o di una decisione di progettazione.
Per esempio, a volte, quando inviamo delle credenziali errate, riceviamo un messaggio che afferma che l'username non è presente nel sistema o la password è errata.
L'informazione ottenuta può essere usata da un attaccante per stilare una lista di utenti esistenti sul sistema.
Queste informazioni possono essere usate per attaccare la web app, per esempio, tramite un brute force o un attacco con username e password di default.

Il tester dovrebbe interagire con il meccanismo di autenticazione dell'app per capire se l'invio di particolari richieste forza l'app a rispondere in modi diversi.
Questo problema esiste perchè le informazioni rivelate dalla web app o dal web server quando l'utente fornisce un'username valida 
sono diverse rispetto a quando ne fornisce una invalida.

In alcuni casi, il messaggio ricevuto rivela se le credenziali fornite sono sbagliate perchè l'username non è valida o la password non lo è.
A volte, i tester possono enumerare gli utenti esistenti inviando un username e una password vuota.

## How to Test

Nel black-box testing, il tester non conosce nulla della specifica app, dell'username, della logica applicativa, dei messaggi di errore nella pagina di login, o delle facility di recovery della password.
Se l'app è vulnerabile, il tester riceve un messaggio di errore che rivela, direttamente o indirettamente, alcune informazioni utili per l'enumerazione degli utenti.

### HTTP Response Message

#### Testing for Valid User/Right Passord

Registra la risposta del server quando invii un user ID valido e una password valida.

> Usando un web proxy, annota le informazioni ricevute a seguito di un'autenticazione avvenuta con successo (risposta HTTP 200, lunghezza della risposta).

#### Testing for Valid User with Wrong Password

Ora, il tester dovrebbe provare a inserire un user ID valido e una password errata e registrare il messaggio di errore generato dall'app.

> Dovrebbero essere mostrati dei messaggi come `Authentication failed.` o `No configuration found.`.
Mentre messaggi come `Login for User foo: invalid password` rivelano l'esistenza dell'utente.
Usando un web proxy, annota le informazioni ricevute a seguito di un'autenticazione fallita (risposta HTTP 200, lunghezza della risposta).

#### Testing for Nonexisting Username

Ora il tester dovrebbe provare a inserire un user ID non valido e una password errata e registrare la risposta del server (il tester dovrebbe essere confidente che l'username non sia valida nell'app).
Registra il messaggio di errore e la risposta del server.

> Potresti ricevere il seguente messaggio `This user is not active` oppure `Login failed for User foo: invalid Account`.
In generale l'app dovrebbe rispondere con lo stesso messaggio di errore e lunghezza a diverse richieste non valide.
Se le risposte non sono le stesse, il tester dovrebbe investigare e trovare la chiave che crea una differenza tra le due risposte.
Per esempio due risposte diverse `The password is not correct` / `User not recognized`.
Le precedenti risposte fanno capire al client che la prima richiesta conteneva un'username valida.
In questo modo può interagire con l'app per recuperare un insieme di user ID validi osservando la risposta.
La seconda risposta del server, fa capire al client che l'username non era valida.
Allo stesso modo può interagire con l'app per stilare un insieme di user ID validi osservando la risposta.

### Other Ways to Enumerate Users

I tester possono enumerare gli utenti in diversi modi, come:

#### Analyzing the Error Code Received on Login Pages

Alcune web app rilasciano uno specifico codice o messaggio di errore che può essere analizzato.

#### Analyzing URLs and URLs Re-directions

Per esempio:

- `http://www.foo.com/err.jsp?User=baduser&Error=0`
- `http://www.foo.com/err.jsp?User=gooduser&Error=2`

Come si vede sopra, quando il tester fornisce un user ID e una password alla web app, vede nell'URL un messaggio indicante che si è verificato un errore.
Nel primo caso ha fornito un user ID non valido e una password errata.
Nel secondo caso, un user ID valido e una password errata, in questo modo viene identificato l'user ID valido.

#### URI Probing

A volte un web server risponde diversamente se riceve una richiesta per una directory esistente o meno.
In alcuni portali ogni utente è associato a una directory.
Se i tester accedono a una directory esistente potrebbero ricevere un errore dal web server.

Alcuni errori comuni ricevuti dai web server sono:

- 403 Forbidden
- 404 Not found

Esempi:

- `http://www.foo.com/account1` - ricevi dal web server: 403 Forbidden
- `http://www.foo.com/account2` - ricevi dal web server: 404 Nof Found

Nel primo caso l'utente esiste, ma il tester non può vedere la pagina,
nel secondo caso l'utente non esiste.
Collezionando queste informazioni il tester può enumerare gli utenti.

#### Analyzing Web Page Titles

I tester possono ricevere informazioni utili nei title delle pagine, 
in cui possono ricevere uno specifico codice o messaggio di errore che rivela se il problema è nell'username o nella password.

Per esempio, se un utente non può autenticarsi sull'app potrebbe riceve una pagina il cui titolo è simile ai seguenti:

- `Invalid user`
- `Invalid authentication`

#### Analyzing a Message Received from a Recovery Facility

Quando usiamo una facility di recovery (es. funzione di password dimenticata) un'app vulnerabile potrebbe restituire un messaggio che rivela se un username esiste o meno.

Per esempio, due messaggi simili ai seguenti:

- `Invalid username: e-mail address is not valid or the specified user was not found.`
- `Valid username: Your password has been successfully sent to the email address you registered with.`

#### Friendly 404 Error Message

Quando richiediamo un utente per una directory che non esiste, non sempre riceviamo un codice di errore 404 Not Found.
Invece, potremmo ricevere un 200 OK con un'immagine, in questo caso possiamo assumere che quando riceviamo quella particolare immagine l'utente non esista.
Questa logica può essere applicata ad altre risposte del web server; 
questo trucco è una buona anlisi dei messaggi del web server e della web app.

#### Analyzing Response Times

Come l'analisi del contenuto delle risposte, deve essere considerato anche il tempo di risposta.
In particolare quando la richiesta forza l'interazione con un servizio esterno (come l'invio di un'email di recupero password), 
ciò può aggiungere diverse centinaia di millisecondi alla risposta, che possono essere usati per determinare se l'utente richiesto esiste.

### Guessing Users

In alcuni casi gli user ID sono creati secondo specifiche policy dell'amministratore o dell'azienda.
Per esempio possiamo vedere un user ID creato in modo sequenziale:

```
CN000100
CN000101
...
```

A volte gli username sono creati con un alias del REALM e un numero sequenziale:

- R1001 - utente 001 del REALM1
- R2001 - utente 001 del REALM2

Per l'esempio precedente possiamo creare un semplice script che compone gli user ID e invia le richieste con un tool come wget per automatizzare le web query in modo da individuare gli user ID validi.
Per creare lo script possiamo anche usare Perl e CURL.

Altre possibilità sono:

- user ID associati ai numeri di carta di credito, o in genere numeri con un pattern
- user ID associati ai nomi reali, es. se Freddie Mercury ha l'user ID "fmercury", puoi presupporre che Roger Taylor abbia l'user ID "rtaylor"

Inoltre, potremmo indovinare l'username dalle informazioni recuperate da una query LDAP o dai motori di ricerca, per esempio, per uno specifico dominio.

> L'enumerazione degli account può causare il lock out degli account dopo un numero predefinito di probe falliti (in base alla policy dell'app).
Inoltre, a volte, il tuo indirizzo IP potrebbe essere bannato da regole dinamiche dell'application firewall o dall'IDS.

### Gray-Box Testing

#### Testing for Authentication Error Messages

Verifica che l'app risponda nello stesso modo a ogni richiesta che generi un'autenticazione fallita.
Per questo problema il black box e il gray box testing hanno lo stesso concetto di base sull'analisi dei messaggi o dei codici di errore ricevuti dalla web app.

> L'app dovrebbe rispondere nello stesso modo per ogni tentativo fallito di autenticazione.
Per esempio: `Credentials submitted are not valid`

## Tools

- OWASP Zed Attack Proxy
- CURL
- Perl

## Remediation

Assicurati che l'app restituisca messaggi di errore generici in risposta ad account, a password o ad altre credenziali utente non validi durante il processo di login.
Assicurati che gli account di sistema e di test di default siano stati eliminati prima del rilascio del sistema in produzione (o in reti non fidate).