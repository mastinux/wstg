# WSTG-ATHN-08

# Testing for Weak Security Question Answer

## Summary

Spesso definite domande e risposte "segrete", le domande e le risposte di sicurezza sono spesso usate per recuperare le password dimenticate (vedi [Testing for Weak Password Change or Reset Functionalities](./WSTG-ATHN-09.md)) o per sicurezza aggiuntiva alla password stessa.

Sono di solito generate durante la creazione dell'account, l'utente deve scegliere da una lista pre-generata e deve fornire una risposta.
Potrebbe essere concessa la generazione di coppie domanda-risposta personalizzate.
Entrambi i metodi sono propensi a insicurezze.
Idealmente, le domande di sicurezza dovrebbero generare risposte che solo l'utente conosce, e non sono indovinabili o scopribili da nessun altro.
In realtà è più difficile di quanto sembri.
Le domande e le risposte di sicurezza si basano sulla segretezza della risposta.
Le domande e le risposte dovrebbero essere scelte in modo che le risposte siano conosciute dal solo proprietario dell'account.
Comunque, anche se molte domande potrebbero non essere note pubblicamente, la maggior parte delle domande che i siti implementano richiedono risposte che sono pseudo-private.

### Pre-generated Questions

La maggior parte delle domande pre-generate sono abbastanza semplicistiche e possono portare alla creazione di risposte insicure.
Per esempio:

- le risposte potrebbero essere conosciute dai membri famigliari o dagli amici stretti dell'utente;
es. 
"Qual è il nome da nubile di tua madre?", 
"Qual è la tua data di nascita?"
- le risposte potrebbero essere facilmente indovinabili;
es. 
"Qual è il tuo colore preferito?",
"Qual è la tua squadra preferita?"
- le risposte potrebbero essere brute forcible;
es. 
"Qual è il nome della tua professoressa preferita?" - la risposta è probabilmente presente in una lista di nomi comuni scaricabile, e quindi può essere preparato un semplice attacco di brute force
- la risposta potrebbe essere scopribile pubblicamente;
es.
"Qual è il tuo film preferito?" - la risposta potrebbe essere trovata sul profilo dei social media dell'utente

### Self-generated Questions

Il problema nel far scegliere agli utenti le proprie domande sta nel fatto che scelgono domande molto insicure, o trascurano l'importanza di avere domande di sicurezza efficaci.
Ecco alcuni esempi di domande reali che mostrano il problema:

- "Quanto fa 1+1?"
- "Qual è il tuo username?"
- "My password is S3curlty!"

## How to Test

### Testing for Weak Pre-generated Questions

Prova a ottenere una lista di domande di sicurezza creando un nuovo account o seguendo il processo di "I don't remember my password".
Prova a generare quante più domande possibili per farti un'idea del tipo di domande di sicurezza che vengono chieste.
Se una domanda rientra nelle categorie descritte prima, è vulnerabile agli attacchi elencati.

### Testing for Weak Self-generated Questions

Prova a creare le domande di sicurezza creando un nuovo account o configurando il recupero password del tuo account esistente.
Se il sistema consente all'utente di generare le proprie domande di sicurezza, allora è vulnerabile alla creazione di domande insicure.
Se il sistema usa domande di sicurezza create dall'utente durante le funzionalità di recupero password e gli username possono essere enumerati (vedi [Testing for Account Enumeration and Guessable User Account](./WSTG-IDNT-04.md)), allora per il tester dovrebbe essere facile enumerare un certo numero di domande create dall'utente.
Dovresti trovare diverse domande deboli usando questo metodo.

### Testing for Brute-forcible Answers

Usa i metodi descritti in [Testing for Weak Lock Out Mechanism](./WSTG-ATHN-03.md) per verificare se l'inserimento di un certo numero di risposte di sicurezza errate scatena il meccanismo di lockout.

La prima cosa da tenere in considerazione quando si cerca di sfruttare le domande di sicurezza è il numero di domande a cui bisogna dare risposta.
La maggior parte delle app richiede la risposta a una sola domanda, mentre alcune app critiche potrebbero richiedere all'utente due o più domande.

Il passo successivo da valuatare è la robustezza delle domande di sicurezza.
La risposta può essere ottenuta da una semplice ricerca Google o con un semplice attacco di social engineering?
Ecco una semplice procedura per lo sfruttamento del sistema delle domande di sicurezza:

- l'app consente all'utente di scegliere le domande a cui bisogna dare risposta?
Se sì, concentrati sulle domande che hanno:

	- una risposta "pubblica"; 
	per esempio qualcosa che può essere trovato con una semplice query sui motori di ricerca
	- una risposta riguardante dei fatti come la "prima scuola" o altri fatti che possono essere cercati
	- poche risposte possibili, come "qual è la tua prima auto".
	Queste domande possono corrispondere a una lista corta di risposte, e sulla base delle statistiche l'attacccante potrebbe scegliere le risposte più probabili

- determina quanti tentativi sono accettati

	- il processo di recupero password consente tentativi illimitati?
	- è presente un periodo di lockout dopo X risposte non corrette?
	Tieni presente che un sistema di lockout può essere esso stesso un problema di sicurezza, dato che può essere sfruttato dall'attaccante per lanciare un DoS contro gli utenti legittimi
	- scegli la domanda appropriata sulla base dell'analisi dei punti precedenti, e cerca di determinare le risposte più probabili

La chiave per sfruttare e raggirare lo schema di domande di sicurezza deboli è di trovare una domanda o un insieme di domande per le quali è facile trovare la risposta.
Cerca sempre le domande che ti danno la maggiore probabilità statistica di trovarela risposta corretta, se sei completamente insicuro di qualsiasi risposta.
Alla fine, uno schema di domande di sicurezza è robusto quanto la domanda più debole.