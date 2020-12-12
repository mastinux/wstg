# WSTG-CONF-11

# Test Cloud Storage

## Summary

I servizi di cloud storage consentono alle web app e ai web service di memorizzare e accedere agli oggetti nello storage.
Una configurazone di access control non adeguata potrebbe comporare l'esposizione di informazioni sensibili, la modifica o l'accesso non autorizzato.

Un noto esempio è la mal configurazione di Amazon S3 bucket, anche se gli altri servizi di cloud storage potrebbero essere esposti ugualmente allo stesso rischio.
Di default, tutti gli S3 bucket sono privati e possono essere acceduti solo dagli utenti a cui è stato concesso l'accesso.
Gli utenti possono concedere l'accesso pubblico all'intero bucket o ai singoli oggetti memorizzati al suo interno.
Ciò potrebbe consentire a un utente non autorizzato di caricare nuovi file, modificare o leggere file memorizzati.

## Test Objectives

Valutare se la configurazione di access control del servizio di cloud storage è adeguata.

## How to Test

Prima identifica le URL per accedere ai dati nell servizio di storage, e poi valuta i seguent test:

- lettura non autorizzata di dati
- caricamento di un nuovo file

Potresti usare curl per i test con i seguenti comandi e vedere se posson essere eseguite azioni non autorizzate.

Per verificare la lettura di un oggetto:

`$ curl -X GET https://<cloud-storage-service>/<object>`

Per verificare l'upload di un file:

`$ curl -X PUT -d 'test' https://<cloud-storage-service>/test.txt`

### Testing for Amazon S3 Bucket Misconfiguration

Le URL degli Amazon S3 bucket possono avere un formato virtual host style o path-style:

- Virtual Hosted Style Access

`https://bucket-name.s3.Region.amazonaws.com/key-name`

Nel seguente esempio, `my-bucket` è il nome del bucket, `us-west-2` è la regione, e `puppy.png` è il nome dell'oggetto':

`https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png`

- Path-Style Access

`https://s3.Region.amazonaws.com/bucket-name/key-name`

Come prima, nel seguente esempio, `my-bucket` è il nome del bucket, `us-west-2` è la regione, e `puppy.png` è il nome dell'oggetto':

`https://s3.us-west-2.amazonaws.com/my-bucket/puppy.jpg`

Per alcune regioni, può essere usato un endpoint globale legacy che non specifica un endpoint regionale.
Anche il suo formato può essere virtual hosted style o path-style:

- Virtual Hosted Style Access

`https://bucket-name.s3.amazonaws.com`

- Path-Style Access

`https://s3.amazonaws.com/bucket-name`

### Identify Bucket URL

Per il black-box testing, le URL S3 possono essere trovate nei messaggi HTTP.
Il seguente esempio mostra un URL di un bucket in un tag `img` restituito in una risposta HTTP.

```xml
...
<img src="https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png">
...
```

Per il gray-box testing, puoi recuperare le URL del bucket dalla web interface di Amazon, dai documenti, dal codice sorgente, o da qualsiasi altra risorsa disponibile.

### Testing with AWS CLI Tool

Oltre al testing con curl, puoi eseguire i test anche con il tool AWS Command Line Interface (CLI).
In questo caso viene usato il protocollo `s3://`.

#### List

Il seguente comando elenca tutti gli oggetti del bucket quando è configurato in modo pubblico:

`aws s3 ls s3://<bucket-name>`

#### Upload

Il seguente comando carica un file:

`aws s3 cp arbitrary-file s3://bucket-name/path-to-save`

Questo esempio mostra il risultato quando l'upload è avvenuto con successo:

```sh
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload: ./test.txt to s3://bucket-name/test.txt
```

Questo esempio mostra il risultato quando l'upload è fallito:

```sh
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload failed: ./test2.txt to s3://bucket-name/test2.txt An error occurred (AccessDenied) when calling the PutObject operation: Access Denied
```

#### Remove

Il seguente comando rimuove un oggetto

`aws s3 rm s3://bucket-name/object-to-remove`

## Tools

- AWS Command Line Interface