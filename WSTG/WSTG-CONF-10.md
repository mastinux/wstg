# WSTG-CONF-10

# Test for Subdomain Takeover

## Summary

Lo sfruttamento di questo tipo di vulnerabilità consente a un attaccante di prendere il controllo del subdomain della vittima.
Per realizzare questo attacco è necessario che:

- il subdomain record del DNS server esterno della vittima punti a una risorsa/servizio esterno/endpoint non esistente o non attivo.
La diffusione di prodotti Anything as a Service (XaaS) e di cloud service pubblici offre molti target potenziali
- il service provider che ospita la risorsa/servizio esterno/endpoint non gestisce adeguatamente la verifica della proprietà dei subdomain

Se il subdomain takeover ha successo sono applicabili diversi attacchi (diffusione di contenuto malevolo, phishing, furto di cookie di sessioni utente, furto di credenziali, etc.).
Questa vulnerabilità può essere sfruttata contro diversi DNS resource record: A, CNAME, MX, NS, etc.
In termini di severity un NS subdomain takeover (anche se meno probabile) ha l'impatto più alto 
dato che un attacco avvenuto con successo può portare al controllo completo di un'intera DNS zone e del dominio della vittima.

### Example1 - GitHub

1. la vittima (victim.com) usa GitHub per lo sviluppo e configura un DNS record (coderepo.victim.com) per accedervi
2. la vittima decide di migrare il repository da GitHub su una piattaforma commerciale
e non rimuove coderepo.victim.com dal suo DNS server
3. un attaccante scopre che coderepo.victim.com è ospitato su GitHub e usa le GitHub Pages per richiedere coderepo.victim.com usando il suo account GitHub

### Example2 - Expired Domain

1. la vittima (victim.com) possiede un altro dominio (victimotherdomain.com) e usa un CNAME record (www) per referenziare l'altro dominio (`www.victim.com` -> `victimotherdomain.com`)
2. a un certo punto, victimotherdomain.com scade ed è disponibile per essere registrato da chiunque.
Dato che il CNAME record non viene calcellato dalla DNS zone di victim.com, 
chiunque registri victimotherdomain.com ha il pieno controllo su www.victim.com finchè il DNS record è presente.

## How to Test

### Black-Box Testing

Il primo passo è enumerare i DNS server della vittima e i suoi resource record.
Ci sono molti modi per farlo per esempio tramite 
DNS enumeration usando una lista di subdomain comuni, 
DNS brute force o 
usando i motori di ricerca e altre risorse OSINT.

Usando il comando dig il tester cerca le seguenti risposte del DNS server che dovrebbero essere analizzate più approfonditamente:
NXDOMAIN,
SERVERFAIL,
REFUSED,
no servers could be reached.

#### Testing DNS A, CNAME Record Subdomain Takeover

Applica una DNS enumeration sul dominio della vittima (victim.com) usando dnsrecon:

```sh
$ ./dnsrecon.py -d victim.com
[*] Performing General Enumeration of Domain: victim.com
...
[-] DNSSEC is not configured for victim.com
[*]
A subdomain.victim.com 192.30.252.153
[*]
CNAME subdomain1.victim.com fictioussubdomain.victim.com
...
```

Identifica quali DNS resource record sono scaduti e puntano a servizi non attivi/non usati.
Usa il comando dig per i CNAME record:

```sh
$ dig CNAME fictioussubdomain.victim.com
; <<>> DiG 9.10.3-P4-Ubuntu <<>> ns victim.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 42950
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
```

Le risposte DNS con status NXDOMAIN sono di nostro interesse.

Per verificare gli A record il tester esegue una ricerca nel database di whois e identifica GitHub come service provider:

```sh
$ whois 192.30.252.153 | grep "OrgName"
OrgName: GitHub, Inc.
```

Il tester visita subdomain.victim.com o invia una richiesta GET che restituisce una risposta "404 - File not found".
Questo risultato è una chiara indicazione della vulnerabilità.

Il tester usa il dominio usando le GitHub Pages, impostando il domain name in `Custom domain`.

#### Testing NS Record Subdomain Takeover

Identifica tutti i nameserver per il dominio in scope:

```sh
$ dig ns victim.com +short
ns1.victim.com
nameserver.expireddomain.com
```

In questo esempio fittizio il tester tramite una ricerca basata su domain registrar controlla se il dominio expireddomain.com è attivo.
Se il dominio può essere acquistato allora il subdomain è vulnerabile.

Le risposte DNS con status SERVFAIL/REFUSED sono di nostro interesse.

### Gray-Box Testing

Il tester ha a disposizione il file di DNS zone, quindi la DNS enumeration non è necessaria.
La metodologia di test è la stessa a quella del Black-Box Testing.

## Remediation

Per mitigare il rischio di subdomain takeover 
i DNS resource record vulnerabili dovrebbero essere rimossi dalla DNS zone.
Il monitoring continuo e i controlli periodici sono raccomandati come best practice.

## Tools

- dig
- recon-ng
- theHarvester
- Sublist3r
- dnsrecon
- OWASP Amass DNS enumeration