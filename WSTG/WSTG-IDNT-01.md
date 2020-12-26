# WSTG-IDNT-01

# Test Role Definitions

## Summary

Nelle aziende moderne è comune definire i ruoli di sistema per gestire gli utenti e l'autorizzazione per l'accesso alle risorse del sistema.
Nella maggior parte delle implementazioni ci sono almeno due ruoli, gli amministratori e gli utenti normali.
I primi rappresentano un ruolo che dà accesso a funzionalità e informazioni privilegiate e sensibili,
i secondi rappresentano un ruolo che dà accesso a funzionalità e informazioni di business di base.
I ruoli ben implementati dovrebbero essere allineati con i processi di business che sono supportati dall'app.

È importante ricorare che l'autorizzazione rigida non è l'unico modo per gestire l'accesso agli oggetti di sistema.
In ambienti più fidati in cui la confidentiality non è un requisito critico,
i controlli più morbidi 
come il flusso dell'app e i log di audit 
possono rispettare i requisiti di data integrity ma non restringere l'accesso dell'utente a funzionalità
o creare strutture di ruoli complessi che sono difficili da gestire.
È importante considerare la regola d'oro nella progettazione dei ruoli, secondo cui
definendone pochi con uno scope ampio (esponendo funzionalità di cui l'utente non ha bisogno)
è da considerarsi negativo come definirne troppi su misura del signolo utente (riducendo l'accesso a funzionalità di cui l'utente ha bisogno).

## Test Objectives

Verificare se i ruoli di sistema definiti nell'app definiscono e separano sufficientemente ogni ruolo di sistema e di business 
per gestire l'accesso appropriato alle funzionalità e alle informazioni del sistema.

## How to Test

Con o senza l'aiuto degli sviluppatori o degli amministratori del sistema, crea una role versus permission matrix.
Questa matrice dovrebbe enumerare tutti i ruoli che possono essere assegnati e i permessi che possono essere applicati agli oggetti, oltre a qualsiasi vincolo presente.
Se la matrix viene fornita con l'app, dovrebbe essere verificata dal tester, 
se non esiste, il tester dovrebbe generarla e determinare se la matrix soddisfa le access policy richieste dall'app.

### Example 1

|ROLE|PERMISSION|OBJECT|CONSTRAINTS|
|-|-|-|-|
|Administrator|Read|Customer records||
|Manager|Read|Customer records|Only records related to business units|
|Staff|Read|Customer records|Only records associated with customers assigned to Manager|
|Customer|Read|Customer record|Only own record|

Un esempio reale di definizioni di ruoli può essere trovato nella role documentation di WordPress.
WordPress ha sei ruoli di default che vanno da Super Admin a Subscriber.

### Example 2

Accedi con i permessi di admin e accedi a pagine che sono solo per gli amministratori.
Poi, accedi come un utente normale e prova ad accedere alle URL di quelle pagine degli amministratori.

1. login come amministratore
2. visita una pagina di amministrazione, es. `http://targetSite/Admin`
3. logout
4. login come utente normale
5. visita la pagina di amministrazione `http://targetSite/Admin`

## Tools

Anche se l'approccio più preciso e accurato per eseguire questo test è quello manuale, 
i tool di spidering sono ugualmente utili.
Accedi con un ruolo per volta e fai lo spidering dell'app (non dimenticare di escludere il link di logout dallo spidering).

## Remediation

Le remediation delle issue possono essere le seguenti:

- Role Engineering
- mapping dei ruoli di business con i ruoli di sistema
- Speration of Duties