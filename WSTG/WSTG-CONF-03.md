# WSTG-CONF-03

# Test File Extensions Handling for Sensitive Information

## Summary

Le estensioni dei file di solito sono usate nei web server per determinare quali tecnologie, linguaggi e plugin devono essere usati per soddisfare la richiesta.
Anche se questo comportamento rispetta gli RFC e i Web Standards, 
l'uso di estensioni di file standard fornisce all'attaccante informazioni utili sulle tecnologie sottostanti usate in una web app e semplifica molto il compito di determinare lo scenario di attacco da usare contro particolari tecnologie.
In aggiunta, le mis-configurazioni dei web server potrebbero rivelare facilmente informazioni confidenziali sulle credenziali di accesso.

La verifica delle estensioni è spesso ustata per validare i file da caricare, che possono condurre a risultati inattesi dato che il contenuto non è quello atteso, o perchè il SO non gestisce adeguatamente quell'estensione.

La determinazione del modo in cui i web server gestiscono le richieste corrispondenti a file aventi estensioni diverse 
potrebbe aiutare a capire il comportamento del web server sul tipo di file che vengono acceduti.
Per esempio, può aiutare a capire quali estensioni di file sono restituite come testo 
rispetto a quelle che vengono eseguite lato server.
Le seconde indicano che le tecnologie, i linguaggi o i plugin sono usati dal web server o dall'app server, e potrebbero fornire informazioni aggiuntive su come la web app è stata progettata.
Per esempio l'estensione ".pl" è di solito associata al supporto Perl lato server.
Comunque, la sola estensione di file potrebbe essere fuorviante e non pienamente conclusiva.
Per esempio, le risorse Perl lato server potrebbero essere rinonimate per nascondere il fatto che sono invece relative a Perl.

## How to Test

### Forced Browsing

Invia richieste con diverse estensioni di file e verifica come sono gestite.
La verifica dovrebbe essere implementata sulla base delle directory.
Verifica le directory che consentono l'esecuzione di script.
Le directory web possono essere identificate tramite tool di scansione che cercano directory note.
Inoltre, la replica della struttura del sito web permette al tester di ricostruire l'albero delle directory web servite dall'app.

Se l'architettura della web app è load-balanced, è importante valutare tutti i web server attivi.
Potrebbe essere più o meno facile, in base alla configurazione dell'infrastruttura di load-balancing.
In un'infrastruttura con componenti ridondanti ci potrebbero essere lievi variazioni nella configurazione dei singoli web server o app server.
Ciò potrebbe succedere se l'architettura web adotta tecnologie eterogenee (pensa all'insieme di web server IIS e Apache in una configurazione di load balancing, che potrebbe introdurre lievi comportamenti asimmetrici tra loro, e probabilmente vulnerabilità diverse).

#### Example

Il tester ha identificato l'esistenza del file `connection.inc`.
Accedendovi direttamente ottiene il suo contenuto:

```php
<?
	mysql_connect("127.0.0.1", "root", "password")
		or die("Could not connect");
?>
```

Il tester determina l'esistenza di un DBMS di backend MySQL e le credenziali deboli usate dalla web app per accedervi.

Le seguenti estensioni di file non dovrebbero essere mai restituite da un web server, dato che sono relative a file che potrebbero contenere informazioni sensibili o a file che non hanno motivo di essere disponibili:

- .asa
- .inc
- .config

Le seguenti estensioni di file sono relative a file che, quando acceduti, sono restituiti o scaricati dal browser.
Quindi, i file con queste estensioni devono essere controllati per verificare se effettivamente devono essere disponibili (e non siano stati lasciati lì per errore), e che non contengano informazioni sensibili:

- .zip, .tar, .gz, .tgz, .rar, ...: file arhivio (compressi)
- .java: non ci sono ragioni per fornire l'accesso a file sorgente Java
- .txt: file di testo
- .pdf: documenti PDF
- .doc, .rtf, .xls, .ppt, ...: documenti Office
- .bak, -old, e altre estensioni che indicano file di backup (per esempio: `~` per i file di backup Emacs)

La precedente lista riporta solo pochi esempi, dato che le estensioni dei file sono troppe per essere riportate completamente.
Fai riferimento a https://filext.com per un database di estensioni completo.

Per identificare i file che hanno una data estensione si può adottare un insieme di tecniche.
Queste possono includere i vulnerability scanner, i tool di spidering e di mirroring, l'ispezione manuale dell'app (in questo modo si superano le limitazioni dello spidering automatico), l'interrogazione di motori di ricerca (vedi [Testing: Spidering and googling](./WSTG-INFO-01.md)).
Vedi anche [Testing for Old, Backup and Unreferenced Files](./WSTG-CONF-04.md) che tratta le issue di sicurezza riguardanti i file "dimenticati".

### File Upload

Il file handling legacy di Windows 8.3 può essere usato per raggirare i filtri di file upload.

Esempio:

```
file.phtml gets processed as PHP code

FILE~1.PHT is served, but not processed by the PHP ISAPI handler

shell.phPWND can be uploaded

SHELL~1.PHP will be expanded and returned by the OS shell, then processed by the PHP ISAPI handler
```

### Gray-Box Testing

L'esecuzione di white-box testing sulla gestione di nomi di file equivale a 
verificare la configurazione del web server o dell'app server e verificare come gestiscono le diverse estensioni di file.

Se la web app si basa su un'infrastruttura eterogenea e load-balanced, questa verifica può dover far fronte a comportamenti diversi.

### Tools

I vulnerability scanner, come Nessus e Nikto controllano l'esistenza di directory web note.
Potrebbero consentire al tester di scaricare la struttura del sito web, che è d'aiuto quando si prova a determinare la configurazione delle directory web e come le singole estensioni di file sono restituite.
Possono essere usati altri tool per questo scopo, tra cui:

- wget
- curl
- google "web mirroring tools"