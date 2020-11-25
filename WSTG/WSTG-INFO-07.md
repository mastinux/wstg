# WSTG-INFO-07

# Map Execution Path Through Application

## Summary

Prima di partire con i test di sicurezza, è fondamentale capire la struttura dell'app.
Senza una conoscenza approfondita della struttura dell'app, è difficile testarla in modo appropriato.

## Test Objectives

Mappare l'app target e capire i principali flussi di esecuzione.

## How to Test

Col black-box testing è estremamente difficile testare tutto il codice.
Non solo perchè il tester non ha la visione dei path di codice interni all'app, 
ma anche se ce li avesse, sarebbe dispendioso in termini di tempo andare a testare tutti i path possibili.
Un compromesso è documentare quello che è stato scoperto e testarlo.

Ci sono diversi modi per il testing e la verifica della copertura del codice:

- Path - testare tutti i path dell'app includendo un'analisi dei valori per ogni decision path.
Anche se questo approccio consente un'analisi approfondita, il numero di path da testare cresce esponialmente a ogni ramo decisionale
- Data Flow (o Taint Analysis) - testare i valori assegnati alle variabili tremite interazioni esterne (normalmente gli utenti).
Si focalizza sul mapping del flusso, sulla trasformazione e sull'uso dei dati tramite l'app
- Race - testare più istanze concorrenti dell'app manipolando gli stessi dati

Il trade off su quale metodo va usato e a che livello approfondire l'analisi dovrebbe essere negoziato con il proprietario dell'app.
Un approccio alternativo potrebbe essere chiedere al proprietario dell'app quali sono le funzioni o le sezioni di codice particolarmente preoccupanti e come è possibile raggiungere queste porzioni di codice.

### Black-Box Testing

Per dimostrare la code coverage al proprietario dell'app, il tester può usare un foglio di calcolo e documentare tutti i link scoperti tramite lo spidering dell'app (manualmente o automaticamente).
Il tester può analizzare con attenzione i decision point nell'app e investigare su quanto codice significativo viene scoperto.
Questo dovrebbe poi essere documentato nel foglio di calcolo con gli URL, la descrizione e gli screenshot dei path scoperti.

### Gray-Box or White-Box Testing

Con il gray-box e il white-box testing è più facile assicurare una code coverage sufficiente.
Le informazioni individuate dal tester soddisferanno i requisiti minimi di code coverage.

#### Example

##### Automatic Spidering

Lo spidering automatico è automatizza la scoperta di nuove URL su un particolare sito.
Il relativo tool parte da una lista di URL da visitare.

ZAP offre le seguenti funzionalità di spidering automatico, che possono essere selezionate a seconda dei bisogni del tester:

- Spider Site - la lista di seed contiene tutte le URI esistenti già trovate per il sito scelto
- Spider Subtree - la lista di seed contiene tutte le URI esistenti già trovate e presenti nel subtree del nodo selezionato
- Spider URL - la lista di seed contiene solo le URI che corrispondono al nodo selezionato (nel Site Tree)
- Spider all in Scope - la lista di seed contiene tutte le URI che il tester ha aggiunto allo scope

## Tools

- ZAP