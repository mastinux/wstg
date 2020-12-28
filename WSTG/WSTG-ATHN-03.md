# WSTG-ATHN-03

# Testing for Weak Lock Out Mechanism

## Summary

I meccanismi di lockout sono usati per mitigare gli attacchi di brute force contro le password.
Gli account sono di solito bloccati dopo 3 o 5 tentativi di login errati e possono essere sbloccati dopo un periodo di tempo predefinito, tramite un meccanismo di sblocco automatico, o tramite l'intervento dell'amministratore.
I meccanismi di lockout richiedono un bilanciamento tra l'accesso non autorizzato e la protezione dell'utente da accessi autorizzati non consentiti.

Nota che questo test dovrebbe affrontare tutte le situazioni in cui i meccanismi di lockout dovrebbero essere applicati, es. quando all'utente vengono poste le domande di sicurezza durante il processo di recupero password (vedi [Testing for Weak Security Question Answer](./WSTG-ATHN-08.md)).

Senza un meccanismo forte di lockout, l'app potrebbe essere suscettibile ad attacchi di brute force.
Dopo un attacco di brute force avvenuto con successo, un utente malevolo potrebbe accedere a:

- informazioni o dati confidenziali:
sezioni private di una web app potrebbero rivelare documenti confidenziali, dati del profilo utente, informazioni findanziarie, dettagli bancari, relazioni tra gli utenti, ecc.
- pannelli amministrativi:
queste sezioni sono usate dai webmaster per gestire (modificare, cancellare, aggiungere) il contenuto della web app, gestire il provisioning degli utenti, assegnare privilegi agli utenti, ecc.
- opportunità per altri attacchi:
le sezioni autenticate di una web app possono contenere vulnerabilità che non sono presenti nella sezione pubblica della web app e possono contenere funzionalità avanzate non disponibili agli utenti pubblici

## Test Objectives

1. valutare la capacità del meccanismo di lockout di mitigare gli attacchi di brute force
2. valutare la resistenza del meccanismo di unlock da parte di account non autorizzati

## How to Test

Di solito, per testare la forza di meccanismi di lockout, devi avere accesso a un account che puoi bloccare.
Se hai solo un account per accedere, esegui questo test alla fine della tua sessione per evitare che tu non possa continuare il testing a causa del lockout dell'account.

Per valutare la capacità del meccanismo di lockout di mitigare gli attacchi di brute force, prova a eseguire un accesso errato usando una password errata per un certo numero di volte, prima di usare la password corretta per verificare che l'account sia effettivamente bloccato.
Un esempio potrebbe essere il seguente:

1. prova ad accedere 3 volte con la password errata
2. accedi con la password corretta, ciò domostra che il meccanismo di lockout non scatta dopo 3 tentativi errati
3. prova ad accedere 4 volte con la password errata
4. accedi con la password corretta, ciò dimostra che il meccanismo di lockout non scatta dopo 4 tentativi errati
5. prova ad accedere 5 volte con la password errata
6. accedi con la password corretta.
L'app restituisce "Your account is locked out", ciò dimostra che il meccanismo di lockout scatta dopo 5 tentativi errati
7. prova ad accedere dopo 5 minuti.
Se l'app restituisce "Your account is locked out", ciò dimostra che il meccanismo di lockout non sblocca automaticamente l'account dopo 5 minuti
8. prova ad accedere dopo 10 minuti.
Se l'app restituisce "Your account is locked out", ciò dimostra che il meccanismo di lockout non sblocca automaticamente l'account dopo 10 minuti
9. accedi dopo 15 minuti.
Ciò dimostra che il meccanismo di lockout sblocca automaticamente l'account dopo 15 minuti

Un CAPTCHA può ostacolare gli attacchi di brute force, ma può anche avere le sue vulnerabilità e non dovrebbe essere usato per rimpiazzare completamente il meccanismo di lockout.

Per valutare la resistenza del meccanismo di lockout allo sblocco non autorizzato dell'account, usa il meccanismo di sblocco e cerca eventuali vulnerabilità.
Un tipico meccanismo di sblocco può utilizzare le domande segrete o un link di sblocco inviato per email.
Il link di sblocco dovrebbe essere un one-time link unico, in modo da impedire a un attaccante di indovinare o fare il replay del link ed eseguire attacchi di brute force.
Le domande e le risposte di sicurezza dovrebbero essere forti (guarda [Testing for Weak Security Question Answer](./WSTG-ATHN-08.md)).

Nota che il meccanismo di sblocco dobrebbe essere usato solo per lo sblocco degli account.
Non per il meccanismo di recupero della password.

Fattori da considerare durante l'implementazione del meccanismo di lockout dell'account sono:

- qual è il rischio di un attacco brute force contro l'app? 
- un CAPTCHA è sufficiente per mitigare il rischio?
- viene usato un meccanismo di lockout lato client (es. JavaScript)?
(Se sì, disabilita il codice lato client durante il test)
- numero di login falliti prima del lockout.
Se la soglia di lockout è troppo bassa 
allora gli utenti validi potrebbero essere bloccati troppo spesso.
Se la soglia di lockout è troppo alta 
allora un attaccante può eseguire più tentativi di brute force prima di essere bloccato.
In base allo scopo dell'app, di solito la soglia si aggira tra i 5 e i 10 tentativi falliti.
- come vengono sbloccati gli account?
	- manualmente dall'amministratore:
	è il metodo di sblocco più sicuro, ma potrebbe essere sconveniente per gli utenti e sottrarre tempo "utile" all'amministratore
		- nota che l'amministratore dovrebbe avere un metodo di recovery nel caso in cui il suo account venisse bloccato
		- questo meccanismo di sblocco potrebbe generare una situazione di DoS se l'obiettivo dell'attaccante fosse di bloccare tutti gli utenti dell'app
	- dopo un intervallo di tempo:
	Quanto dura il lockout?
	È sufficiente per la protezione dell'app?
	Es. una durata dai 5 ai 30 minuti potrebbe essere un buon compromesso tra la mitigazione di attacchi di brute force e il limite creato agli utenti validi
	- tramite un meccanismo self-service:
	come detto prima, questo meccanismo self-service deve essere abbastanza sicuro da impedire che un attaccante possa sbloccare gli account da solo

## Remediation

Applica i meccanismi di sblocco in base al livello di rischio.

In ordine crescente di sicurezza:

- lockout e sblocco time-based
- sblocco self-service (invia un'email di sblocco all'indirizzo email registrato)
- sblocco manuale dell'amministratore
- sblocco manuale dell'amministratore con identificazione dell'utente