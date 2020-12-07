# WSTG-CONF-05

# Enumerate Infrastructure and Application Admin Interfaces

## Summary

Le interfacce amministrative potrebbero essere presenti nell'app o nell'app server.
Queste consentono ad alcuni utenti di esegure attività privilegiate sul sito.
Bisogna esegure dei test per rilevare se e come queste funzionalità privilegiate possono essere accedute da un utente non autorizzato o standard.

Un'app potrebbe richiedere all'interfaccia amministrativa di abilitare un utente privilegiato ad accedere alle funzionalità che potrebbero cambiare il modo in cui il sito funziona.
Queste modifiche possono essere:

- gestione degli account
- design e layout del sito
- manipolazione dei dati
- modifiche nella configurazione

In molte istanze, tali interfacce non hanno controlli sufficienti per proteggerle da accessi non autorizzati.
Il testing mira a individuare queste interfacce amministrative e ad accedere alle funzionalità predisposte per gli utenti privilegiati.

## How to Test

### Black-Box Tesing

La seguente sezione descrive i vettori che possono essere usati per testare la presenza di interfacce amministrative.
Queste tecniche possono anche essere usate per testare i problemi relativi a privilege escalation, e sono descritte più avanti in questa guida (per esempio [Testing for bypassing authorization schema](./WSTG-ATHZ-02.md) e [Testing for Insecure Direct Object References](./WSTG-ATHZ-04.md)).

- enumerazione di directory e di file.
Potrebbe essere presente un'interfaccia amministrativa ma non visibie al tester.
Il tentativo di indovinare il path dell'interfaccia amministrativa potrebbe essere semplice: `/admin` o `/administrator` etc. o in alcuni scenari può essere individuata tramite i Google dorks
- ci sono molti tool disponibili per eseguire il brute forcing dei contenuti del server,
per maggiori informazioni guarda la sezione Tools che segue.
Un tester dovrebbe anche identificare il nome della pagina di amministrazione.
Accedendo alla pagina identificata potrebbe portare all'interfaccia
- i commenti e i link nel codice sorgente.
Molti siti usano un codice comune che è caricato per tutti gli utenti.
Esaminando tutto il codice inviato al client, potresti individuare i link alle funzionalità amministrative
- verifica la documentazione del server e dell'app.
Se l'app server o la stessa app sono deployed con le loro configurazioni di default potrebbe essere possibile accedere all'interfaccia amministrativa usando le informazioni descritte nella documentazione della configurazione o di aiuto.
Le liste di password di default dovrebbero essere consultate quando si individua un'interfaccia amministrativa e questa richiede le credenziali
- informazioni disponibili pubblicamente.
Molte app come wordpress hanno interfacce amministrative di default
- porta alternativa del server.
Le interfacce amministrative potrebbero essere attive su una porta diversa rispetto a quella dell'app principale.
Per esempio, l'interfaccia amministrativa di Apache Tomcat può essere trovata di solito sulla porta 8080
- modifica dei parametri.
Un parametro GET o POST o una variabile cookie potrebbero essere richiesti per abilitare le funzinalità amministrative.
Indizi a riguardo possono essere l'inclusione di campi nascosti:

```xml
<input type="hidden" name="admin" value="no">
```

o di valori dei cookie:

```
Cookie: session_cookie; useradmin=0
```

Dopo aver individuato un'interfaccia amministrativa, dovrebbe essere usata una combinazione delle precedenti tecniche per cercare di raggirare l'autenticazione.
Se non ci si riesce, si può applicare un brute force attack.
In questo caso il tester deve essere a conoscienza di un eventuale lockout dell'account amministrativo.

### Gray-Box Tesing

Dovrebbe essere eseguito un'esame più dettagliato dei componenti del server e dell'app per assicurare l'hardening (es. le pagine amministrative non sono accedibili da chiunque grazie all'IP filtering o ad altri controlli), 
e dove applicabile, la verifica che tutti i componenti non usino credenziali o configurazioni di default.
Il codice sorgente dovrebbe essere controllato per assicurarsi che il modello di autenticazione e autorizzazione assicuri una chiara separation of duties tra gli utenti normali e gli amministratori del sito.
Le funzioni dell'UI condivise tra utenti normali e amministrativi dovrebbero essere controllate per assicurare una chiara separazione tra l'utilizzo di questi componenti e l'eventuale information leakage derivante dall'uso di queste funzionalità condivise.

Ogni web framework potrebbe avere le sue pagine o i suoi path amministrativi di default.
Per esempio:

WebSphere:

```
/admin
/admin-authz.xml
/admin.conf
/admin.passwd
/admin/*
/admin/logon.jsp
/admin/secure/logon.jsp
```

PHP:

```
/phpinfo
/phpmyadmin/
/phpMyAdmin/
/mysqladmin/
/MySQLadmin
/MySQLAdmin
/login.php
/logon.php
/xmlrpc.php
/dbadmin
```

FrontPage:

```
/admin.dll
/admin.exe
/administrators.pwd
/author.dll
/author.exe
/author.log
/authors.pwd
/cgi-bin
```

WebLogic:

```
/AdminCaptureRootCA
/AdminClients
/AdminConnections
/AdminEvents
/AdminJDBC
/AdminLicense
/AdminMain
/AdminProps
/AdminRealm
/AdminThreads
```

WordPress:

```
wp-admin/
wp-admin/about.php
wp-admin/admin-ajax.php
wp-admin/admin-db.php
wp-admin/admin-footer.php
wp-admin/admin-functions.php
wp-admin/admin-header.php
```

## Tools

- OWASP ZAP - Forced browse è il successore del progetto DirBuster
- THC-HYDRA è un tool che consente il brute-forcing di molte interfacce, 
inclusa l'autenticazione HTTP form-based
- è molto meglio usare un brute forcer con un buon dizionario, per esempio quello di [netsparker](https://www.netsparker.com/blog/web-security/svn-digger-better-lists-for-forced-browsing/)