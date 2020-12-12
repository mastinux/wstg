# WSTG-CONF-09

# Test File Permission

## Summary

Quando a una risorsa vengono assegnati dei permessi che le forniscono accesso a un numero maggiore di risorse rispetto a quelle strettamente necessarie,
si potrebbero esporre informazioni sensibili, 
o le risorse accedute potrebbero essere modificate.
Ciò è particolarmente pericoloso quando la risorsa è relativa a una configurazione, a un'esecuzione o a dei dati utente.

Un esempio è un file eseguibile che viene eseguito da utenti non autorizzati.
Un altro esempio, le informazioni di un account o il valore di un token per accedere a un API - sempre più presenti nei moderni web service e microservice - possono essere memorizzati in un file di configurazione i cui permessi sono impostati su world-readable nell'installazione di default.
Queste informazioni sensibili possono essere esposte 
verso un malicious actor interno dell'host 
o verso un attaccante remoto che ha compromesso il servizio con altre vulnerabilità ma le ha ottenute con i privilegi di un utente normale.

## How to Test

In Linux, usa il comando `ls` per controllare i permessi di un file.
In alternativa, puoi usare `namei` per elencare i permessi di un file.

`$ namei -l /PathToCheck/`

I file e le directory che necessitano di una verifica sui permessi includono ma non sono limitati a:

- file/directory web
- file/directory di configurazione
- file/directory sensibili (dati cifrati, password, chiavi)
- file/directory di log (security log, operation log, admin log)
- file eseguibili (script, EXE, JAR, class, PHP, ASP)
- file/directory di database
- file/directory temporanei
- file/directory di upload

## Tools

- Windows AccessEnum
- Windows AccessChk
- Linux namei

## Remediation

Imposta i permessi di file e directory in modo da evitare che gli utenti non autorizzati possano accedere a risorse critiche.