# WSTG-IDNT-05

# Testing for Weak or Unenforced Username Policy

## Summary

I nomi degli account utente hanno spesso una struttura predefinita 
(es. l'account di Joe Bloggs è jbloggs e quello di Fred Nurks è fnurks), quindi i nomi di account validi possono essere facilmente indovinati.

## Test Objectives

Determina se la struttura del nome dell'account rende l'app vulnerabile all'enumerazione degli account.
Determina se i messaggi di errore dell'app consentono l'enumerazione degli account.

## How to Test

- determina la struttura dei nomi degli account
- controlla la risposta dell'app ad account validi e non validi
- usa le diverse risposte a nomi di account validi e non validi per enumerare i nomi di account validi
- usa dizionari di nomi di account per enumerare i nomi di account validi

## Remediation

Assicurati che l'app restituisca messaggi di errore generici in risposta 
a nomi di account, 
a password o 
ad altre credenziali inserite durante il processo di login.