# WSTG-IDNT-02

# Test User Registration Process

## Summary

Alcuni siti web offrono un processo di user registration che automatizza (o semi-automatizza) il processo di provisioning all'utente per l'accesso al sistema.
I requisiti d'identità per l'accesso variano dall'identificazione avvenuta con successo a nessun requisito, in base ai requisiti di sicurezza del sistema.
Molte app pubbliche automatizzano completamente il processo di registrazione dato che la quantità di utenti da verificare rende impossibile la gestione manuale.
Tuttavia, molte app aziendali potrebbero prevedere la creazione manuale di nuovi utenti, quindi questo test potrebbe non essere applicabile.

## Test Objectives

1. verifica che i requisiti di identità per la registrazione utente siano allineati con i requisiti di business e di sicurezza
2. valida il processo di registrazione

## How to Test

Verifica che i requisiti di identità per la registrazione dell'utente siano allineati con i requisiti di business e di sicurezza:

- chiunque può registrarsi per l'accesso?
- le registrazioni sono validate da un umano prima del provisioning, 
o sono concesse automaticamente se i criteri sono rispettati?
- la stessa persona o identità può registrarsi più volte?
- gli utenti possono registrarsi per diversi ruoli o permessi?
- quele prova di identità è richiesta per completare la registrazione?
- le identità registrate sono verificate?

Verifica il processo di registrazione:

- le informazioni sull'identità possono essere facilmente forged o falsificate?
- lo scambio di informazioni sull'identità può essere manipolato durante la registrazione?

### Example

In WordPress, l'unico requisito di identificazione è un indirizzo email che è accessibile a chi si registra.

Al contrario, in Google i requisiti di registrazione includono il nome, la data di nascita, lo Stato, il numero mobile, l'indirizzo email e la risposta CAPTCHA.
Anche se solo due di questi possono essere verificati (l'indirizzo email e il numero mobile), i requisiti di registrazione sono più stringenti di WordPress.

## Tools

Un proxy HTTP può essere utile per verificare questo controllo.

## Remediation

Implementa i requisiti di identificazione e di verifica che corrispondono ai requisiti di sicurezza delle informazioni che le credenziali proteggono.