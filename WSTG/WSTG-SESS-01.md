# WSTG-SESS-01

# Testing for Session Management Schema

## Summary

Uno dei componenti principali di qualsiasi app web-based è il meccanismo tramite cui controlla e mantiene lo stato di un utente che interagisce con essa.
Per evitare l'autenticazione continua per ogni pagina del sito o del servizio, le web app implementano diversi meccanismi per memorizzare e validare le credenziali per un lasso di tempo pre-determinato.
Questi meccanismi sono conosciuti come Session Management.

In questo test, il tester vuole controllare che i cookie e gli altri token di sessione siano creati in modo sicuro e non predicibile.
Un attaccante che è in grado di predirre e forgiare un cookie debole può facilmente impersonare le sessioni di utenti legittimi.

I cookie sono usati per implementare la session management e sono descritti in dettaglio nell'RFC 2965.
In poche parole, quando un utente accede a un'app che deve tenere traccia delle azioni e dell'identità di quell'utente tra più richieste, un cookie (o più cookie) è generato dal server e inviato al client.
Il client invierà il cookie verso il server in tutte le connessioni successive fino a che il cookie scade o viene distrutto.
I dati memorizzati nel cookie possono fornire al server un grande spettro di informazioni su chi è l'utente, quali azioni ha eseguito, quali sono le sue prefernze, etc. fornendo quindi uno stato al protocollo stateless come HTTP.

Un esempio tipico è un carrello di un sito di shopping online.
Attraverso la sessione di un utente, l'app deve tenere traccia della sua identità, del suo profilo, dei prodotti che ha scelto di comprare, della quantità, dei prezzi singoli, degli sconti, etc.
I cookie sono un modo efficiente per memorizzare e passare questa informazione tra server e client (gli altri metodi sono i parametri dell'URL e i campi nascosti).

Data l'importanza dei dati che memorizzano, i cookie sono quindi vitali per la sicurezza complessiva dell'app.
Essere in grado di manipolare i cookie può consentire 
di fare l'hijacking delle sessioni di utenti legittimi, 
di ottenere privilegi più alti nelle sessioni attive,
e in generale di influenzare le operazioni dell'app in modo non autorizzato.

In questo test il tester deve controllare se i cookie rilasciati ai client possono resistere a una serie di attacchi mirati a interferire con le sessioni di utenti legittimi e con la stessa app.
L'obiettivo generale è essere in grado di forgiare un cookie che sarà considerato valido dall'app e che fornirà una qualche forma di accesso non autorizzato (session hijacking, privilege escalation, ...).

Di solito i passi principali del pattern di attacco sono i seguenti:

- cookie collection: raccolta di un numero sufficiente di esempi di cookie
- cookie reverse engineering: analisi dell'algoritmo di generazione del cookie
- cookie manipulation: creazione di un cookie valido per eseguire l'attacco.
Questo ultimo passo potrebbe richiedere un gran numero di tentativi, in base a come il cookie è creato (cookie brute-force attack)

Un altro pattern di attacco consiste nell'overflowing di un cookie.
In senso stretto, questo attacco ha una natura diversa, dato che i tester non provano a ricreare un cookie perfettamente valido.
Invece, l'obiettico è l'overflow di un'area di memoria, in modo da interferire con il comportamento corretto dell'app e possibilmente iniettando (ed eseguendo remotamente) codice malevolo.

## How to Test

### Black-Box Testing and Examples

Tutte le interazioni tra client e app dovrebbero essere testate rispetto ai seguenti criteri:

- le direttive Set-Cookie sono taggate come Secure?
- le operazioni sui cookie avvengono su connessioni non cifrate?
- il cookie può essere forzato su connessioni non cifrate?
- in caso positivo, l'app come assicura la sicurezza?
- ci sono cookie persistenti?
- che valore viene assegnato a Expire per i cookie persistenti, ed è un valore ragionevole?
- i cookie che dovrebbero essere transitori sono configurati come tali?
- quali impostazioni di Cache-Control HTTP/1.1 sono usate per proteggere i cookie?
- quali impostazioni di Cache-Control HTTP/1.0 sono usate per proteggere i cookie?

#### Cookie Collection

Il primo passo richiesto per manipolare il cookie è di capire come l'app crea e gestisce i cookie.
Per questo compito, i tester devono provare a rispondere alle seguenti domande:

- quanti cookie sono usati dall'app? 

Naviga l'app.
Annota quando i cookie vengono creati.
Fai una lista dei cookie ricevuti, la pagina che li imposta (con la direttiva Set-Cookie), il dominio per i quali sono validi, il loro valore, e le loro caratteristiche.

- quali parti dell'app genera o modifica i cookie?

Navigando l'app, trova quali cookie restano costanti e quali sono modificati.
Quali eventi modificano i cookie?

- quali parti dell'app richiede questo cookie per essere accedute e utilizzate?

Individiua quali parti dell'app hanno bisogno di un cookie.
Accedi a una pagina, poi prova di nuovo senza il cookie, o con un suo valore modificato.
Prova a mappare quali cookie sono usati e dove.

Un foglio di calcolo che mappa ogni cookie con le parti dell'app corrispondenti e le informazioni relative può essere un output di valore di questa fase.

#### Session Analysis

I session token (Cookie, SessionID, campi nascosti) dorebbero essere esaminati per assicurare la loro qualità da un punto di vista di sicurezza.
Dovrebbero essere controllati rispetto a dei criteri come la loro randomness, resistenza ad analisi statistica e crittografica e information leakage.

- Struttura del token e information leakage

Il primo passo è esaminare la struttura e il contenuto di un session ID fornito dall'app.
Un errore comune è includere dati specifici nel token, invece di rilasciare un valore generico e riferirsi ai dati reali solo lato server.

Se il Session ID è in chiaro, la struttura e i dati pertinenti potrebbero essere immediatamente ovvi come l'esempio che segue:

`192.168.100.1:owaspuser:password:15:58`

Se parte del token sembra essere cifrato o hashed, dovrebbe essere confrontato con diverse tecniche per verificare tecniche di offuscamento banali.
Per esempio la string `192.168.100.1:owaspuser:password:15:58` è rappresentata in Hex, Base64 e MD5 come segue:

- Hex `3139322E3136382E3130302E313A6F77617370757365723A70617373776F72643A31353A3538`
- Base64 `MTkyLjE2OC4xMDAuMTpvd2FzcHVzZXI6cGFzc3dvcmQ6MTU6NTg=`
- MD5 `01c2fc4f0a817afd8366689bd29dd40a`

Dopo aver identificato il tipo di offuscamento, potrebbe essere possibile risalire ai dati originali.
In molti casi, comunque, è improbabile.
Anche se, potrebbe essere utile individuare l'encoding utilizzato dal formato del messaggio.
Inoltre, se sia il formato che la tecnica di obfuscation possono essere dedotti, possono essere impiegati attacchi brute-force.

I token ibridi possono includere informazioni come indirizzi IP o User ID insieme a una porzione codificata, come la seguente:

`owaspuser:192.168.100.1:a7656fafe94dae72b1e1487670148412`

Dopo aver analizzato un singolo token, bisogna esaminare il campione rappresentativo.
Tramite una semplice analisi dei token si potrebbero rilevare immediatamente pattern ovvi.
Per esempio, un token di 32 bit potrebbe includere 16 bit di dati statici e 16 bit di dati variabili.
Ciò potrebbe indicare che i primi 16 bit rappresentano un attributo fisso dell'utente, es. l'username o l'indirizzo IP.
Se i secondi 16 bit incrementano secondo un intervallo regolare, potrebbe essere l'indizio della generazione del token sequenziale o time-based.
Vedi gli esempi.

Se nei token vengono identificati degli elementi statici, bisognerebbe raccogliere ulteriori campioni, cambiando un potenziale input per volta.
Per esempio, l'accesso attraverso un user o un indirizzo IP diverso potrebbe portare alla variazione di porzioni statiche del token.

Bisognerebbe verificare le seguenti aree durante il testing della struttura del Session ID:

- quali parti del Session ID sono statiche?
- quali informazioni confidenziali sono memorizzate in chiaro nel Session ID? es. username/UID, indirizzo IP
- quali informazioni confidenziali facilmente decodificate sono memorizzate?
- quali informazioni possono essere dedotte dalla struttura del Session ID?
- quali porzioni del Session ID sono statiche per le stesse condizioni di login?
- quali pattern ovvi sono presenti nel Sesssion ID?

#### Session ID Predictability and Randomness

L'analisi delle aree variabili (se ci sono) del Session ID dovrebbe essere intrapresa per stabilire l'esistenza di pattern riconoscibili o predicibili.
Queste analisi potrebbero essere eseguite manualmente o tramite tool statistici o crittografici su misura od OTS per dedurre dei pattern nel contenuto dei Session ID.
I controlli manuali dovrebbero essere inclusi per i Session ID rilasciati per le stesse condizioni di login - es. lo stesso username, password e indirizzo IP.

Il tempo è anche un importante fattore che deve essere valutato.
Dovrebbero essere eseguite un numero elevato di connessioni simultanee per raccogliere campioni nella stessa finestra temporale e mantenere la variabile tempo costante.
Anche una quantizzazione di 50 ms o meno potrebbe essere troppo grossolana e un token ottenuto in questo modo potrebbe rilevare componenti time-based che diversamente non verrebbero individuati.

Gli elementi variabili dovrebbero essere analizzati nel corso del tempo per determinare se hanno una natura incrementale.
Quando sono incrementali, i pattern relativi a un tempo assoluto o incrementale dovrebbero essere approfonditi.
Molti sistemi usano il tempo come seed per i propri elementi pseudo-random.
Quando i pattern sono apparentemente random, l'hashing del datetime o altre varianti di ambiente dovrebbero essere presi in considerazione.
Tipicamente, il risultato di un hash crittografico è un numero decimale o esadecimale quindi dovrebbe essere identificabile.

Durante l'analisi delle sequenze di Session ID, pattern o cicli, gli elementi statici o le dipendenze client dovrebbero essere considerate come un possibile elemento contributivo alla struttura e al funzionamento dell'app.

- i Session ID sono random in modo provabile?
I valori risultanti possono essere riprodotti?
- le stesse condizioni di input producono lo stesso ID in esecuzioni successive?
- i Session ID sono resistenti ad analisi statica o criptoanalisi?
- quali elementi del Session ID sono time-linked?
- quale porzione del Session ID è predicibile?
- il prossimo ID può essere dedotto, conoscendo l'algoritmo di generazione e i precedenti ID?

#### Cookie Reverse Engineering

Ora che il tester ha enumerato i cookie e ha un'idea generale del loro uso, è il momento di approfondire i cookie che sembrano interessanti.
A quali cookie il tester è interessato?
Un cookie, per offrire un metodo sicuro di session management, deve combinare diverse caratteristiche, ognuna delle quali è destinata a proteggere il cookie da forme diverse di attacchi.

Queste caratteristiche sono riassunte di seguito:

- Unpredictability: 
un cookie deve contenere dei dati difficili da indovinare.
Più è difficile forgiare un cookie valido, più è difficile accedere a una sessione di un utente valido.
Se un attaccante è in grado di indovinare il cookie usato in una sessione attiva di un utente legittimo, sarà in grado di impersonare quell'utente (session hijacking).
Per rendere un cookie non predicibile, possono essere usati dei valori casuali o la crittografia
- Tamper resistance:
un cookie deve esesere resistente a tentativi malevoli di modifica.
Se un tester riceve un cookie come IsAdmin=no, è banale modificarlo per ottenere i diritti amministrativi, a meno che l'app esegua un doppio controllo (per esempio, concatenando al cookie un hash cifrato del suo valore)
- Expiration:
un cookie critico deve essere valido solo per un periodo di tempo appropriato e deve essere cancellato dal disco o dalla memoria per evitare il rischio che venga riutilizzato.
Ciò non si applica ai cookie che memorizzano dati non critici che devono essere ricordati tra più sessioni (es. look-and-feel del sito)
- "Secure" flag:
un cookie il cui valore è critico per l'integrità della sessione dovrebbe avere questo flag abilitato per consentire la sua trasmissione solo su un canale cifrato per impedire le intercettazioni

Qui l'approccio è di collezionare un numero sufficienti di istanze di un cookie e cercare pattern nei suoi valori.
Il significato esatto di "sufficiente" può variare da una manciata di campioni, se il metodo di generazinoe di cookie è molto facile da spezzare, a diverse centinaia, se il tester ha bisogno di eseguire delle analisi matematiche.

È importante prestare particolare attenzione al workflow dell'app, dato che lo stato della sessione può avere un pesante impatto sui cookie collezionati.
Un cookie collezionato prima di essere autenticato può essere molto diverso da un cookie ottenuto dopo l'autenticazione.

Un altro aspetto da tenere in considerazione è il tempo.
Memorizza sempre il momento esatto in cui il cookie è stato ottenuto, quando c'è la possibilità che il tempo giochi un ruolo nel valore del cookie (il server potrebbe usare un time stamp come perte del valore del cookie).
Il tempo registrato potrebbe essere l'orario locale o il time stamp sul server incluso nella risposta HTTP (o entrambi).

Durante l'analisi dei valori collezionati, il tester dovrebbe provare a immaginare tutte le variabili che potrebbero influenzare il valore del cookie e provare a variarli nel tempo.
Inviare versioni modificate del cookie al server può essere utile a capire come l'app legge ed elabora il cookie.

Esempi di controlli che possono essere eseguiti in questa fase:

- quale character set è usato nel cookie?
il cookie ha un valore numerico?
alfanumerico?
esadecimale?
cosa succede se il tester inserisce nel cookie un carattere che non appartiene al charset che il server si aspetta?
- il cookie è composto da sottoparti diverse che contengono pezzi diversi di informazioni?
le diverse parti come sono separate?
con quali delimitatori?
alcune parti del cookie potrebbero avere un'alta varianza, altre potrebbero essere costanti, altre potrebbero assumere solo un insieme limitato di valori.
Il primo passo fondamentale è dividere il cookie nei suoi componenti base

Un esempio di cookie con una struttura facilmente deducibile è il seguente:

`ID=5a0acfc7ffeb919:CR=1:TM=1120514521:LM=1120514521:S=j3am5KzC4v01ba3q`

Questo esempio mostra cinque diversi campi, che trasportano diversi tipi di dati:

- ID esadecimale
- CR small integer
- TM e LM large integer 
(curiosamente contengono lo stesso valore; vale la pena vedere cosa succede modificando uno dei due)
- S alfanumerico

Anche se non ci sono dei delimitatori, può essere d'aiuto avere più campioni.
Come esempio, diamo un'occhiata alla seguente serie:

`0123456789abcdef`

#### Brute Force Attacks

Gli attacchi di brute force portano inevitabilmente a domande relative alla predicibilità e alla randomness.
La varianza del Session ID deve essere considerata insieme alla durata della sessione e ai time-out dell'app.
Se la varianza del Session ID è relativamente piccola, e la validità del Session ID è lunga, la probabilità di successo di un attaccco di brute force è molto alta.

Un session ID lungo (o almeno uno con una grande varianza) e un periodo di validità più corto dovrebbero rendere più difficile il successo di un attacco di brute force.

- quanto dovrebbe durare un attacco di brute-force su tutti i possibili Session ID?
- lo spazio dei Session ID è abbastanza grande per impedire gli attacchi di brute force?
Per esempio, la lunghezza della chiave è sufficientemente lunga se confrontata con il periodo di validità?
- i ritardi tra tentativi di connessione con diversi Session ID mitigano il rischio di quest'attacco?

### Gray-Box Testing and Examples

Se il tester ha accesso all'implementazione dello schema di session management, può eseguire i seguenti controlli:

- random session token

Il Session ID o il Cookie rilasciato al client non dovrebbe essere facilmente predicibile (non usare algoritmi lineari basati su valori predicibili come l'indirizzo IP del client).
Si suggerisce l'uso di algoritmi crittografici con una chiave di lunghezza di 256 bit.

- token length

Il Session ID avrà una lunghezza di almeno 50 caratteri.

- session time-out

I token di session dovrebbero avere un time-out definito (in base alla criticità dei dati gestiti dall'app)

- cookie configuration

	- non-persistent: solo in RAM
	- secure (solo su canali HTTPS): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; secure`
	- HTTPOnly (non leggibile da uno script): `Set-Cookie: cookie=data; path=/; domain=.aaa.it; HttpOnly`

## Tools

- OWASP ZAP
- Burp Sequencer
- YEHG's JHijack