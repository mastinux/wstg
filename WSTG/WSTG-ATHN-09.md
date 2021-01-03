# WSTG-ATHN-09

# Testing for Weak Password Change or Reset Functionalities

## Summary

La funzione di cambio e reset password di un'app è un meccanismo self-service per gli utenti.
Questo meccanismo consente agli utenti di cambiare o reimpostare velocemente la propria password senza l'intervento dell'amministratore.
Quando le password vengono cambiate di solito lo sono all'interno dell'app.
Quando le password sono reimpostate sono renderizzaate all'interno dell'app o inviate tramite email.
Ciò potrebbe indicare che le password sono memorizzate in chiaro o sono in un formato decifrabile.

## Test Objectives

1. determina la resistenza dell'app al raggiro del processo di cambio account finalizzato al cambio password di un altro account
2. determina la resistenza della funzionalità di reset password contro i tentativi e i raggiri

## How to Test

Sia per il cambio e che per il reset password è importante controllare che:

- se gli utenti, oltre agli amministratori, possono cambiare o reimpostare le password di altri account
- se gli utenti possono manipolare o raggirare il processo di cambio o di reset password per cambiare o reimpostare la password di un altro utente o dell'amministratore
- se il processo di cambio o reset password è vulnerabile al CSRF

### Test Password Reset

Oltre ai precedenti controlli è importante verificare anche:

- quali informazioni sono richieste per il reset password?  
Il primo passo è verificare se le domande di sicurezza sono richieste.
L'invio della password (o del link di reset password) all'indirizzo email dell'utente senza chiedere una domanda di sicurezza significa fidarsi al 100% di quell'indirizzo email, che non è adatto alle app che necessitano di un alto livello di sicurezza.
Dall'altra parte, se vengono chieste le domande di sicurezza, il passo successivo è valuatare la loro robustezza.
Questo specifico test è discusso in [Testing for Weak Security Question Answer](./WSTG-ATHN-08.md).
- il reset delle password come viene comunicato all'utente?
Lo scenario più insicuro è se il tool di reset password mostra la password;
ciò permette all'attaccante di accedere con lo specifico account, e a meno che l'app fornisca informazioni sull'ultimo login la vittima non si accorge che il suo account è stato compromesso.
Uno scenario meno insicuro è se il tool di reset password forza l'utente a cambiare immediatamente la password.
Anche se non è stealthy come nel primo caso, consente all'attaccante di ottenere l'accesso e di buttare fuori l'utente effettivo.  
Si ha il migliore livello di sicurezza se il reset password è fatto inviando un'email all'indirizzo con cui l'utente si è inizialmente registrato;
ciò forza l'attaccante non solo a indovinare a quale indirizzo email il reset password è stato inviato (a meno che l'app mostri quest'indirizzo) ma anche a compromettere quell'account email per ottenere la password temporanea o il link di reset password.
- le password di reset sono generate in modo casuale?
Lo scenario meno sicuro è quando l'app invia o mostra la vecchia password in chiaro il che significa che le password non sono memorizzate in formato hash, che è esso stesso un problema di sicurezza.  
Il miglior livello di sicurezza viene raggiunto se le password sono generate in modo casuale con un algoritmo di sicurezza che non può essere spezzato
- la funzionalità di reset password chiede conferma prima di effettuare il cambio?
Per limitare gli attacchi DoS l'app dovrebbe inviare un link tramite email all'utente con un token random, e solo se l'utente visita il link allora la procedura di reset viene completata.
Ciò assicura che la password corrente sarà ancora valida fino a che il reset non è stato confermato

### Test Password Change

Oltre ai passi precedenti è importante verificare:

- la vecchia password viene richiesta per completare il cambio?  
Lo scenario più insicuro è quando l'app consente il cambio password senza richiedere quella corrente.
Infatti se un attaccante è in grado di prendere il controllo di una sessione valida sarebbe in grado di cambiare la password della vittima.
Vedi anche [Testing for Weak Password Policy](./WSTG-ATHN-07.md).

## Remediation

La funzione di cambio o reset password è una funzione sensibile e ha bisogno di alcune protezioni, come chiedere all'utente di riautenticarsi o presentargli uno screen di conferma durante il processo.