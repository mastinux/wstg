# WSTG-CONF-07

# Test HTTP Strict Transport Security

## Summary

L'header HTTP Strict Transport Security (HSTS) è un meccanismo che i siti web hanno per comunicare ai web browser che tutto il traffico scambiato con un dato dominio deve sempre avvenire su https,
ciò impedisce di trasmettere le informazioni su richieste non protette.

Considerando l'importanza di questa misura di sicurezza è fondamentale verificare che il sito web stia usando questo header HTTP, 
per assicurarsi che tutti i dati viaggino in modo cifrato dal web browser al server.

HSTS consente a una web app di informare il browser, tramite l'uso di un header speciale, che non dovrebbe mai stabilire una connessione con lo specifico dominio usando HTTP.
Invece, dovrebbe stabilire automaticamente tutte le richieste di accesso al sito su HTTPS.

L'header in questione usa due direttive:

- `max-age`:
per indicare per quanti secondi il browser dovrà convertire le connessioni da HTTP a HTTPS
- `includeSubDomains`:
per indicare che tutti i sotto domini della web app devono usare HTTPS

Segue un esempio di implementazione dell'header HTST:

`Strict-Transport-Security: max-age=60000; includeSubDomains`

L'uso di questo header dalle web app deve essere verificato per impedire che i seguenti problemi di sicurezza si possano verificare:

- gli attaccanti fanno lo sniffing del traffico di rete 
e accedono alle informazioni trasferite tramite il canale non cifrato
- gli attaccanti sfruttano un MITM grazie all'accettazione di certificati non fidati
- gli utenti che inseriscono erroneamente un indirizzo nel browser usando HTTP invece di HTTPS, 
o gli utenti che cliccano su un link alla web app che è stato riportato con HTTP

## How to Test

La verifica della presenza dell'header HSTS può essere fatta controllando la presenza dell'header HSTS nella risposta del server in un proxy, o usando curl come segue:

```sh
$ curl -s -D- https://owasp.org | grep Strict
Strict-Transport-Security: max-age=15768000
```