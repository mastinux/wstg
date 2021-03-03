# WSTG-ATHZ-04

# Testing for Insecure Direct Object References

## Summary

Un Insecure Direct Object References si verifica quando un'app fornisce accesso diretto agli oggetti in base all'input utente.
Il risultato di questa vulnerabilità è che gli attaccanti possono raggirare l'autorizzazione e accedere a risorse direttamente nel sistema, per esempio a record del database o a file.
L'Insecure Direct Object Reference consente agli attaccanti di raggirare l'autorizzazione e accedere a risorse direttamente modificando il valore del parametro usato per puntare a un oggetto.
Tali risorse possono essere le entry di un database appartenenti ad altri utenti, file del sistema, o altro.
Ciò è causato dal fatto che l'app prende l'input utente e lo usa per recuperare un oggetto senza eseguire sufficienti controlli di autorizzazione.

## How to Test

Per testare questa funzionalità il tester deve prima mappare tutti i punti in cui l'app usa l'input utente per far direttamente riferimento agli oggetti.
Per esempio, i punti in cui l'input utente è usato per accedere a una riga del database, un file, pagine dell'app o altro.
Successivamente il tester dovrebbe modificare il valore del parametro usato per riferirsi all'oggetto e verificare se è possibile recuperare gli oggetti appartenenti agli altri utenti o l'autorizzazione può essere raggirata.

Il miglior modo per verificare la direct object reference è avere almeno due (se non altri) utenti per far riferimento a diversi oggetti e funzioni.
Per esempio, due utenti aventi accesso a oggetti diversi (come informazioni su un acquisto, messaggi privati, etc.) e (se rilevanti) utenti con privilegi diversi (per esempio utenti amministrativi) per vedere se ci sono riferimenti diretti a funzionalità dell'app. 
Avendo più utenti il tester risparmia tempo rispetto a provare a indovinare i nomi di oggetti diversi, dato che conosce già quelli appartenenti agli altri utenti.

Seguono diversi scenari per questa vulnerabilità e i metodi per testarli:

### The Value of a Parameter is Used Directly to Retrieve a Database Record

Esempio:

`http://foo.bar/somepage?invoice=12345`

In questo caso, il valore del parametro `invoice` è usato come indice per la tabella invoices del database.
L'app prende il valore di questo parametro e lo usa in una query al database.
L'app restituisce le informazioni dell'invoice all'utente.

Dato che il valore di `invoice` va direttamente nella query, modificando il valore del parametro è possibile recuperare qualsiasi invoice, indipendentemente dell'utente a cui appartiene.
Per testare questo caso il tester dovrebbe ottenere l'identificatore di un invoice appartenente a un utente diverso (assicurandosi che per business logic non dovrebbe vedere tali informazioni), e poi controllare se è possibile accedere agli oggetti senza autorizzazione.

### The Value of a Parameter is Used Directly to Perform an Operation in the System

Esempio:

`http://foo.bar/changepassword?user=someuser`

In questo caso, il valore del parametro `user` viene usato per dire all'app l'utente del quale si vuole cambiare la password.
In molti casi questo step è parte di un wizard, o un'operazione multi-step.
Nel primo step l'app riceverà una richiesta per l'utente per il quale si vuole cambiare la password, e nello step successivo l'utente fornirà una nuova password (senza richiedere quella corrente).

Il parametro `user` è usato per far riferimento direttamente all'oggetto dell'utente per il quale si vuole cambiare la password.
Per testare questo caso il tester dovrebbe provare a fornire un username diverso rispetto a quello attualmente loggato, e 
verificare se è possibile modificare la password di un altro utente.

### The Value of a Parameter Is Used Directly to Retrieve a File System Resource

Esempio:

`http://foo.bar/showImage?img=img00011`

In questo caso, il valore del parametro `file` è usato per dire all'app quale file l'utente vuole recuperare.
Fornendo il nome o l'identificatore di un file diverso (per esempio file=image00012.jpg) l'attaccante sarà in grado di recuperare oggetti appartenenti a utenti diversi.

Per testare questo caso, il tester dovrebbe ottenere un riferimento a cui l'utente non dovrebbe aver accesso e provare ad accedervi usando il valore nel parametro file.
Nota: questa vulnerabilità è spesso sfruttata insieme a una vulnerabilità di directory/path traversal (vedi [Testing Directory Traversal File Include](./WSTG-ATHZ-01.md)).

### The Value of a Parameter is Used Directly to Access Application Functionality

Esempio:

`http://foo.bar/accessPage?menuitem=12`

In questo caso, il valore del parametro `menuitem` è usato per dire all'app quale oggetto del menu (e quindi quale funzionalità dell'app) l'utente sta provando ad accedere.
Assumi che l'utente abbia un accesso limitato e quindi abbia i link per accedere agli oggetti 1, 2, e 3.
Modificando il valore del parametro `menuitem` è possibile raggirare l'autorizzazione e accedere a funzionalità addizionali dell'app.
Per verificare questo caso il tester 
identifica il punto in cui la funzionalità dell'app è determinata dal riferimento all'oggetto del menu, 
mappa i valori degli oggetti del menu a cui l'utente di test ha accesso, e 
prova ad accedere agli altri oggetti.

Nell'esempio precedente, la modifica di un singolo parametro è sufficiente.
Comunque, a volte il riferimento potrebbe essere diviso tra più di un parametro, e il test dovrebbe essere adattato.