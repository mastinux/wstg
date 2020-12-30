# WSTG-ATHN-07

# Testing for Weak Password Policy

## Summary

Il meccanismo più prevalente e più facilmente gestibilè è una password statica.
La password rappresenta le chiavi del regno, ma le si dà spesso poca attenzione preferendo l'usabilità.
In ognuno dei recenti hack contro profili importanti che hanno rivelato le credenziali degli utenti, le password più comuni erano `123456`, `password` e `qwerty`.

## Test Objectives

Determina la resistenza dell'app contro gli attacchi di brute force usando i password dictionary disponibili e valutando i requisiti di lunghezza, di complessità, di riutilizzo e di invecchiamento delle password.

## How to Test

1. quali caratteristiche sono concesse e vietate per le password?
All'utente è richiesto di usare caratteristiche di tipo diverso come lettere minuscole e maiuscole, numeri e caratteri speciali?
2. ogni quanto l'utente può cambiare la password?
Dopo quanto tempo l'utente può cambiare la password dopo un cambio precedente?
Gli utenti possono raggirare i requisiti di non riutilizzare la stessa password cambiandola 5 volte in modo che l'ultimo cambio corrisponda alla stessa password
3. l'utente quando deve cambiare la password?
Sia il NIST che l'NCSC sconsigliano di forzare la scadenza della password, anche se potrebbe essere richiesta dagli standard come PCI DSS
4. un utente quanto può riusare una password?
L'app mantiene uno storico delle ultime 8 password dell'utente?
5. quanto deve essere diversa la nuova password dall'ultima?
6. viene impedito all'utente di usare il suo username o altre informazioni dell'account (come il suo nome o cognome) nella password?
7. quali sono le lunghezze massima e minima che possono essere usate, e che sono appropriate per la sensibilità dell'account e dell'app?
8. è possibile impostare password comuni come `Password1` o `123456`?

## Remediation

Per mitigare il rischio di password facilmente indovinabili che facilitano l'accesso non autorizzato ci sono due soluzioni:
introdurre controlli di autenticazione addizionali (es. two-factor authentication)
o introdurre una password policy forte.
La più semplice ed economica delle due è l'introduzione di una password policy forte che assicura lunghezza, complessità, riuso e invecchiamento della password adeguati;
anche se idealmente entrambe dovrebbero essere implementate.