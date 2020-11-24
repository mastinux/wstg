# WSTG-INFO-05

# Review Webpage Comments and Metadata for Information Leakage

## Summary

È molto comune, e anche raccomandato, per i programmatori inserire commenti dettagliati e metadati nel loro codice sorgente.
Tuttavia, i commenti e i metadati inclusi nel codice HTML potrebbero rivelare informazioni interne che non dovrebbero essere a disposizione degli attaccanti.
Bisogna rivedere commenti e metadati per determinare se vengono rivelate informazioni sensibili.

## Test Objectives

Rivedere i commenti e i metadati delle pagine web per capire meglio il funzionamento dell'app e verificare eventuali information leakage.

## How to Test

I commenti HTML sono spesso usati dagli sviluppatori per includere informazioni di debug sull'app.
A volte se ne dimenticano e li lasciano nell'ambiente di produzione.
I tester dovrebbero cercare i commenti HTML che iniziano con `<!--`.

### Black-Box Testing

Controlla nel codice sorgente HTML i commenti che contengono informazioni sensibili che possono aiutare l'attaccante ad avere maggiore conoscenza dell'app.
Potrebbe essere codice SQL, username, password, indirizzi IP interni, o informazioni di debug.

```xml
...
<div class="table2">
	<div class="col1">1</div><div class="col2">Mary</div>
	<div class="col1">2</div><div class="col2">Peter</div>
	<div class="col1">3</div><div class="col2">Joe</div>

	<!-- Query: SELECT id, name FROM app.users WHERE active='1' -->
</div>
...
```

Il tester potrebbe trovare qualcosa simile al seguente:

```xml
<!-- Use the DB administrator password for testing:
f@keP@a$$w0rD -->
```

Cerca nelle informazioni sulla versione HTML i numeri di versione e URL di Data Type Definition (DTD).

```xml
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```

- `strict.dtd` - DTD strict di default
- `loose.dtd` - DTD loose
- `frameset.dtd` - DTD per i documenti frameset

Alcuni meta tag non forniscono vettori di attacco attivi ma consentono all'attaccante di profilare l'app:

```xml
<META name="Author" content="Andrew Muller">
```

Un meta tag (non compliant con WCAG) è il refresh.

```xml
<META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">
```

Un uso comune dei meta tag è specificare le keyword che un motore di ricerca può usare per migliorare la qualità dei suoi risultati.

```xml
<META name="keywords" lang="en-us" content="OWASP, security, sunshine, lollipops">
```

Anche se molti web server gestiscono l'indicizzazione per i motori di ricerca tramite il file robot.txt, questa può essere gestita anche tramite i meta tag.
Il seguente tag indica ai motori di ricerca di non indicizzare e non seguire i link contenuti nella pagina HTML.

```xml
<META name="robots" content="none">
```

La Platform for Internet Content Selection (PICS) e il Protocol for Web Description Resources (POWDER) forniscono un'infrastruttura per associare i metadati con i contenuti Internet.

### Gray-Box Testing

Non applicabile.

## Tools

- wget
- funzione "view source" dei browser
- eyeballs
- curl