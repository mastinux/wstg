# WSTG-INFO-10

# Map Application Architecture

## Summary

Un'infrastruttura di più web server interconnessi ed eterogenei 
può includere centinaia di web app e 
rende la gestione e la verifica delle configurazioni un passo fondamentale nel testing e nel deploy di ogni singola app.
Infatti basta una singola vulnerabilità per impattare sulla sicurezza dell'intera infrastruttura, e anche piccoli problemi che sembrano poco importanti potrebbero trasformarsi in rischi gravi per altre app sullo stesso server.

Per risolvere questi problemi, è importantissimo eseguire una revisione approfondita delle configurazioni e delle issue di sicurezza.
Prima di eseguire una revisione approfondiata è necessario mappare la rete e l'architettura dell'app.
I diversi elementi che compongono l'app devono essere individuati per capire come interagiscono con la web app e come incidono sulla sicurezza.

## How to Test

### Map the Application Architecture

L'architettura dell'app deve essere mappata tramite alcuni test per determinare quali componenti sono usati per comporre la web app.
In sistemi piccoli, come una semplice app CGI, potrebbe essere usato un piccolo server per eseguire l'app scritta in C, Perl, o Shell CGI e un eventuale meccanismo di autenticazione.

In un sistema più complesso, come un sistema di banking online, potrebbero essere coinvolti più server.
Questi potrebbero includere un reverse proxy, un web server di frontend, un server applicativo e un database server o un LDAP server.
Ognuno di questi server verrà usato per scopi diversi e potrebbe essere separato in reti diverse con un firewall a loro protezione.
Questa configurazione crea diverse DMZ in modo che in caso di un accesso remoto al web server non sarà possibile accedere ai diversi elementi dell'architettura, dato che sono isolati.

L'ottenimento delle informazioni sull'architettura dell'app può essere facile se sono fornite ai tester dagli sviluppatori tramite documenti o interviste, ma possono essere molto difficili da recuperare in caso di un blind penetration test.

In quest'ultimo caso, il tester partirà dall'assunzione che è di fronte a un sistema semplice (singolo server).
Poi recupererà informazioni da altri test e derivererà i diversi elementi, metterà in dubbio questa assunzione e estenderà la mappa dell'architettura.
Il tester inizierà a farsi semplici domande come: "C'è un firewall che protegge l'app?".
Si darà una risposta a questa domanda in base ai risultati dei network scan sul web server e l'analisi di un eventuale filtering delle porte (non riceve una risposta o riceve un messaggio ICMP unreachable) o se il server è direttamente connesso a Internet (restituisce RST per tutte le porte ceh non sono in ascolto).
Quest'analisi può essere migliorata per determinare il tipo di firewall usato sulla base di test dei pacchetti di rete.
È un firewall statefull o è un filtro tramite access list impostata su un router?
Come è configurato?
Può essere raggirato?

Per individuare un reverse proxy è necessario analizzare il banner del web server, che potrebbe rivelare direttamente l'esistenza di un reverse proxy (per esempio viene restituito `WebSEAL`).
Può essere individuato anche confrontando le risposte ricevute con le risposte che ci si aspetta.
Per esempio, alcuni reverse proxy agiscono da "intrusion prevention system" (o web-shield) bloccando attacchi noti al web server.
Se il web server invece di rispondere con un messaggio 404 a richieste verso una pagina non disponibile 
rispnde con un messaggio di errore diverso per alcuni attacchi web comuni come quelli fatti dagli scanner CGI, 
potrebbe essere un'indicatore della presenza di un reverse proxy (o un application level firewall) che sta filtrando le richieste e restituendo una pagina di errore diversa rispetto a quella che ci si aspetta.
Un altro esempio: se il web server restituisce un insieme di metodi HTTP disponibili (incluso TRACE) ma i metodi invocati restituiscono un errore, allora probabilmente qualcosa li sta bloccando.

In alcuni casi, lo stesso sistema di protezione si rivela:

```
GET /web-console/ServerInfo.jsp%00 HTTP/1.0


HTTP/1.0 200
Pragma: no-cache
Cache-Control: no-cache
Content-Type: text/html
Content-Length: 83

<TITLE>Error</TITLE>
<BODY>
<H1>Error</H1>
FW-1 at XXXXXX: Access denied.</BODY>
```

I reverse proxy possono essere usati anche come proxy-caches per accelerare le performance dei server applicativi di backend.
È possibile individuarli tramite gli header di risposta.
Possono essere individuati anche in base al timing delle richieste che dovrebbero essere cached dal server e confrontando i tempi di risposta rispetto a quelli della prima richiesta.

Un altro elemento che può essere individuato è il load balancer.
Di solito, questi sistemi smistano una connessione TCP/IP su più server in base a diversi algoritmi (round-robin, carico del web server, numero di richieste, etc.).
Quindi, per poter identificare il load balancer è necessario esaminare più richieste e confrontare i risultati per determinare se le richieste sono elaborate dallo stesso server o meno.
Per esempio, in base all'header `Date` se l'orologio dei server non è sincronizzato.
In alcuni casi, il processo di load balancing potrebbe aggiungere infromazioni negli header che sono facilmente individuabili, 
come il cookie `AlteonP` aggiunto dal load balancer Alteon WebSystem di Nortel.

I web server applicativi sono facilmente individuabili.
La richiesta per più risorse è gestita dallo stesso server applicativo (non dal web server) e l'header di risposta cambierà di molto (inclusi valori diversi o aggiuntivi nell'header di risposta).
Un altro modo per identificarli è se il web server cerca di impostare un cookie che è indicativo di un certo web server applicativo (come `JSESSIONID` fornito da alcuni server J2EE) o se cerca di riscrivere le URL per fare session tracking.

Tuttavia, i back end di autenticazione (come LDAP directories, database relazionali, o server RADIUS) non sono facilmente individuabili dall'esterno in un modo diretto, dato che saranno nascosti dall'app stessa.

L'uso di un database di back end può essere determinato semplicemente usando l'app.
Se ci sono contenuti molto dinamici generati "on the fly", è probabile che siano estratti dall'app da qualche sorta di database.
A volte il modo in cui le informazioni sono richieste potrebbero rivelare l'esistenza di un database di backend.
Per esempio, un app di shopping online che usa un identificatore ('id') durante la navigazione tra articoli diversi.
Comunque, durante un blind penetration test, è possibile ottenere informazioni sul database sottostante solo se ci sono delle vunlerabilità nell'app, come inadeguata gestione degli errori o SQL injection.