# WSTG-ATHN-05

# Testing for Vulnerable Remember Password

## Summary

Le credenziali sono la tecnologia di autenticazione più ampiamente usata.
A causa dell'ampio utilizzo della coppia username-password, gli utenti non sono in grado di gestire adeguatamente le proprie credenziali data la moltitudine di app usate.

Per aiutare gli utenti nella gestione delle credenziali, sono nate diverse tecnologie:

- le app forniscono una funzionalità di remember me che permette all'utente di rimanere autenticato per periodi più lunghi, senza dover fornire di nuovo le credenziali
- i password manager (inclusi quelli dei browser) consentono all'utente di memorizzare le proprie credenziali in modo sicuro e più tardi di iniettarle nei form senza l'intervento dell'utente

## How to Test

Dato che questo metodo garantisce una migliore user experience e permette all'utente di dimenticarsi di tutte le sue credenziali, esso aumenta la superficie d'attacco.
Alcune app:

- memorizzano le credenziali codificate nello storage del browser. Verificale con [Testing Browser Storage](./WSTG-CLNT-12.md) e con [Testing for Session Management Schema](./WSTG-SESS-01.md).
Le credenziali non dovrebbero essere memorizzate in alcun modo nell'app client, e dovrebbero essere sostituite dai token generati dal server
- iniettano automaticamente le credenziali dell'utente che possono essere sfruttate da:
	- attacchi di ClickJacking
	- attacchi di CSRF
- i token dovrebbero essere analizzati in termini di tempo di vita, infatti alcuni token non scadono mai e espongono gli utenti al rischio di furto.
Assicurati di eseguire [Testing Session Timeout](./WSTG-SESS-07.md)

## Remediation

- segui le good practice sulla [session management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- assicurati che le credenziali non siano memorizzate in clear text e che non siano facilmente recuperabili in formato codificato o cifrato dai meccanismi di storage del browser;
dovrebbero essere memorizzate sul server seguendo le good practice sul [password storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)