# WSTG-INFO-03

# Review Webserver Metafiles for Information Leakage

## Summary

Questa sezione descrive come analizzare il file robots.txt per identificare information leakage della directory o dei path dei folder della web app.
Inoltre, la lista di directory che devono essere escluse dagli spider/robot/crawler può essere creata tramite il controllo [Map Execution Paths Through Application](./WSTG-INFO-07.md)

## Test Objectives

1. information leakage dei path della directory o del folder della web app
2. creare una lista di directory che devono essere escluse da spider/robot/crawler

## How to Test

### robots.txt

Gli spider, i robot o i crawler web recuperano una web page e poi ricorsivamente seguono gli hyperlink per recuperare altro contenuto web.
Il loro comportamento è specificato dal Robots Exclusion Protocol nel file robots.txt presente nella webroot directory.

Come esempio, l'inizio del file robots.txt da https://www.google.com/robots.txt è il seguente:

```
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
...
```

La direttiva User-agent si riferisce allo specifico spider/robot/crawler.
Per esempio per l'`User-agent: Googlebot` ci si riferisce allo spider di Google
mentre `User-agent: bingbot` si riferisce al crawler di Microsoft/Yahoo!.
`User-agent: *` nell'esempio precedente si applica a tutti gli spider/robot/crawler.

La direttiva `Disallow` specifica quali risorse non sono concesse a spider/robot/crawler.

Gli spider/robot/crawler web possono ignorare intenzionalmente le direttive `Disallow` specificate nel file robots.txt, come quelli dei social network per assicurarsi che i link condivisi siano ancora validi.
Quindi, il file robots.txt non dovrebbe essere considerato un meccanismo per imporre restrizioni su come il contenuto web viene acceduto, memorizzato, o pubblicato da terze parti.

#### robots.txt in Webroot - with wget or curl

Il file robots.txt viene recuperato dalla web root directory del web server.
Per esempio, per recuperare il file robots.txt da `www.google.com` usando `wget` o `curl`:

```sh
$ wget http://www.google.com/robots.txt
$ head -n5 robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
$

$ curl -O http://www.google.com/robots.txt
$ head -n5 robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
$
```

#### robots.txt in webroot - with rockspider

rockspider automatizza la creazione dello scope iniziale di spider/robot/crawler di file e directory/folder di un sito web.

Per esempio, per creare lo scope iniziale basandosi sulla direttiva `Allow` per `www.google.com` usando rockspider:

```sh
$ ./rockspider.pl -www www.google.com
"Rockspider" Alpha v0.1_2

Copyright 2013 Christian Heinrich
Licensed under the Apache License, Version 2.0

1. Downloading http://www.google.com/robots.txt
2. "robots.txt" saved as "www.google.com-robots.txt"
3. Sending Allow: URIs of www.google.com to web proxy i.e. 127.0.0.1:8080
	/catalogs/about sent
	/catalogs/p? sent
	/news/directory sent
	...
4. Done.

$
```

\#FIXME rockspider genera `ERROR: opening stream: can't connect (timeout): Transport endpoint is not connected`

#### Analyze robots.txt Using Google Webmaster Tools

I proprietari dei siti web possono usare la funzione "Analyze robots.txt" di Google per analizzare il sito web come parte dei suoi Google Webmaster Tools.
Questo tool può essere d'aiuto durante il testing e la procedura da seguire è la seguente:

1. accedi al Google Webmaster Tools con un account Google
2. nella dashboard, scrivi l'URL del sito da analizzare
3. scegli tra i metodi disponibili e segui le istruzioni

### META tag

I tag `<META>` si trovano nella sezione HEAD di ogni documento HTML e dovrebbero essere consistenti all'interno di tutto il sito web, per far fronte anche al caso in cui il robot/spider/crawler inizi l'analisi da un link che non si trova nella webroot, es. un deep link.

Se non è presente l'entry `<META NAME="ROBOTS" ...>` il Robots Exclusion Protocol sceglie come valori di default `INDEX, FOLLOW`.
Perciò, le altre due entry valide definite dal Robot Exclusion Protocol sono precedute da `NO...`, es. `NOINDEX` e `NOFOLLOW`.

Gli spider/robot/crawler web possono ignorare intenzionalmente il tag `<META NAME="ROBOTS" ...>` dato che prevale il conenuto del file robots.txt.
Quindi, i tag non dovrebbero essere considerati come un meccanismo primario, piuttosto un controllo complementare al file robots.txt.

#### META tags - with Burp

Sulla base delle direttive `Disallow` presenti nel file robots.txt nella webroot, una regular expression cerca `<META NAME="ROBOTS"` all'interno di ogni web page e confronta i risultati con il contenuto del file robots.txt nella webroot.

Per esempio, il file robots.txt da facebook.com contiene un'entry `Disallow: /ac.php` nel file robots.txt e la ricerca trova `<meta name="robots" content="noodp, noydir" />`.

Potrebbe essere considerato un problema dato che `INDEX,FOLLOW` è il valore di default per il tag `<META>` secondo quanto specificato dal Robots Exclusion Protocol, ma `Disallow: /ac.php` è presente in robots.txt.

## Tools

- Browser
- curl
- wget
- rockspider