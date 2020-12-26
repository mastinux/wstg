# WSTG-IDNT-03

# Test Account Provisioning Process

## Summary

Il provisioning degli account offre all'attaccante la posssibilità di creare un account valido senza l'applicazione degli adeguati processi di identificazione e autorizzazione.

## Test Objectives

Verifica quali account possono fare il provisioning dei altri account e di che tipo.

## How to Test

Determina quali ruoli sono in grado di fare il provisioning di altri utenti e per che tipo di account.

- è presente una verifica, un controllo e un'autorizzazione delle richieste di provisioning?
- è presente una verifica, un controllo e un'autorizzazione delle richieste di de-provisioning?
- un amministratore può fare il provisioning di altri amministratori o solo di altri utenti?
- un amministratore o altri utenti possono fare il provisioning di account con privilegi più alti dei propri?
- un amministratore o un utente può fare il de-provisioning di se stesso?
- come vengono gestite le risorse degli utenti de-provisioned?
Sono eliminate?
L'accesso viene trasferito?

### Example

In WordPress, solo il nome e l'indirizzo email dell'utente sono richiesti per il provisioning dell'utente.

Per il de-provisioning dell'utente, l'amministratore deve scegliere gli utenti, scegliere `Delete` dal dropdown menu e fare l'apply.
Viene poi mostrato un dialog box che chiede cosa fare dei post dell'utente (cancellarli o trasferirli).

## Tools

Anche se l'approccio più completo e accurato è l'esecuzione manuale, i proxy HTTP possono essere utili.