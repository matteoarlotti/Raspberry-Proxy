# Raspberry-Proxy
Ciao Marketers! Oggi realizziamo un proxy Socks5 con l'ausilio di un Raspberry Pi, del vostro router domestico e di qualche linea di codice!

# Occorrente
- Raspberry Pi Model B (versione 2 o 3, indifferente) -> Amazon: http://amzn.to/2g269he
- Router ADSL/Fibra domestico
- Scheda Micro SD (da almeno 4 giga) -> Amazon: http://amzn.to/2hXPJqQ
- Cavo di rete -> Amazon: http://amzn.to/2wHz8tS
- Raspbian Lite -> Download: https://downloads.raspberrypi.org/raspbian_lite_latest
- Etcher -> Download: https://etcher.io/
- Putty (con Mac possiamo usare l'app Terminale) -> Download: http://bit.ly/2y2zyit
- Fing Network Tool (per Android o iPhone sugli store, è free)
- Una tastiera USB, un cavo HDMI, una TV o un monitor con supporto HDMI (ci servirà per il primo avvio)

# Preparazione
Scarichiamo tutto l'occorrente, installiamo Etcher, e scompattiamo Raspbian Lite (il file ce ne ricaveremo sarà un .img

# Configurazione SD
Inseriamo la SD nel nostro PC, lanciamo Etcher, e selezioniamo il file .img precedentemente estratto. 
Il processo non impiegherà più di 5 minuti.

# Accensione Raspberry
Procediamo ora all'avvio del nostro Raspberry. Inseriamo la micro sd nel suo alloggiamento. Colleghiamo il tutto alla tv/monitor e inseriamo la tastiera. Colleghiamo l'alimemtatore microusb ad una porta USB (pc o alimentatore da cellulare), e si parte con il primo avvio. Quando ce lo chiede, logghiamoci con le credenziali di default. 
```sh
login as: pi
password: raspberry
```

# Attivazione SSH
Per rendere la nostra single-board (il Raspberry) operativa al 100% e da remoto (senza tastiera e monitor), dobbiamo attivare una funzione aggiuntiva: SSH.
Rendiamoci amministratori con il codice seguente:
```sh
sudo su
```
Una volta fatto, apriamo la configurazione di sistema con
```sh
raspi-config
```
e diamo invio.
Dovremo scendere nel menù Interfacing Options (freccie Giù e Invio per confermare), SSH, e una volta evidenziato YES dare nuovamente invio. Il sistema ci riporterà alla prima schermata, freccia Destra x 2 volte e porteremo il selettore su Finish. Invio.
Ora digitiamo
```sh
shutdown now
```
e attendiamo che il nostro Raspberry si spenga, siamo pronti per collegarlo vicino al router e non toccarlo mai più :)

# Collegamento Raspberry al router
Grazie alle porte USB presenti nel retro del nostro router possiamo alimentare in modo automatico la nostra board senza dover ricorrere ad altri alimentatori, in caso di assenza invece possiamo utilizzare quello di un vecchio smartphone. Colleghiamo anche il nostro cavo di rete ad una porta libera sul router e lasciamo che il tutto si avvii come in precedenza.
Nel frattempo apriamo l'app FING per scoprire quale indirizzo interno è stato assegnato al nostro Raspberry. Avviamo la scansione di rete attraverso l'icona in alto a destra e cerchiamo nella lista dei risultati l'immagine di un lampone, con nome raspberrypi. L'IP restituito sarà quello che utilizzeremo per amministrare la nostra board.

# Ricapitolando #1
- Abbiamo tutto l'occorrente
- Abbiamo attivato la gestione remota via SSH del Raspberry 
- Abbiamo messo in rete il nostro mini computer
Siamo pronti per proseguire!

# Login Remoto con MAC
Se lavoriamo su MAC possiamo avvalerci dell'app pre-installata Terminale.
Una volta avviata scriveremo questa riga 
```sh
ssh pi@<IP della scheda ottenuto con Fing>
```
esempio: ssh pi@192.168.1.148. Dando invio ci chiederà la password, la stessa di prima: "raspberry"

# Login Remoto con Windows
Avviamo Putty, una applicazione che ci permette di instaurare connessioni SSH.
In Hostname inseriamo l'IP della nostra scheda (esempio: 192.168.1.148) che ci ha dato prima Fing e diamo Invio. Quando ce lo chiede, logghiamoci con le credenziali di default. 
```sh
login as: pi
password: raspberry
```
# Installazione modulo
Iniziamo subito ad installare i moduli necessari, andremo a caricare un software che opererà come Server SOCKS5 e accetterà connessioni in ingresso per poi completarle attraverso la nostra linea domestica, in questo modo, tutti i bot che operano con instagram, crederanno che il traffico da loro creato viene fatto dallo stesso ip della vostra abitazione.
Digitiamo il seguente comando
```sh
sudo apt-get install dante-server
```
quando ci chiederà conferma, clicchiamo il tasto Y della nostra tastiera per far procedere l'installazione.

Ora dovremo provvedere alla configurazione del nostro server. Per prima cosa, eliminiamo la configurazione di default, piena di cose inutili. Per farlo:
```sh
rm /etc/danted.conf
```
Procediamo alla creazione di un nuovo file di configurazione
```sh
nano /etc/danted.conf
```
Si aprirà una schermata totalmente vuota con un cursore che lampeggia. Il file da incollare è il seguente. Utilizzate CTRL+C e CTRL+V come comandi rapidi.
```sh
logoutput: syslog
user.privileged: root
user.unprivileged: nobody

# The listening network interface or address.
internal: 0.0.0.0 port=1080

# The proxying network interface or address.
external: eth0

# socks-rules determine what is proxied through the external interface.
# The default of "none" permits anonymous access.
socksmethod: username

# client-rules determine who can connect to the internal interface.
# The default of "none" permits anonymous access.
clientmethod: none

client pass {
        from: 0.0.0.0/0 to: 0.0.0.0/0
        log: connect disconnect error
}

socks pass {
        from: 0.0.0.0/0 to: 0.0.0.0/0
        log: connect disconnect error
}
```
Spieghiamolo rapidamente: nella prima parte, diciamo al sistema che destinazione usare per i file di log e quale utente sarà l'amministratore. Segue la parte relativa alla connessione: accetta tutte le connessioni dal network locale. Definiamo quale scheda di rete usare per navigare sul web (l'unica che abbiamo: eth0). Richiediamo al proxy di funzionare solo con autentificazione, altrimenti un proxy libero sarebbe nella mani dei pirati informatici nel giro di pochi minuti. Client-rules è un secondo step di verifica, possiamo affidarci al primo blocco via SOCKS, altrimenti usare la voce "clientmethod: username" al posto di "clientmethod: none". A questo punto rendiamo libero la navigazione sugli IP, non sappiamo con quale i nostri bot si collegheranno al proxy, per questo lasciamo tutto aperto.

Incollato il nostro file, clicchiamo CTRL+X per uscire. Ci chiederà se vogliamo salvare il file: Y (tastiera) e INVIO per confermare.
Una volta salvato, possiamo avviare il sistema:
```sh
service danted start
```
Per avere la conferma, digitiamo
```sh
netstat -nlpt | grep dant 
```
e otterremo un risultato analogo, sinonimo di "tutto ok, il server è avviato!"
```sh
 tcp        0      0 0.0.0.0:1080            0.0.0.0:*               LISTEN      21157/danted       
```

Altro test da effettuare è una prova di navigazione attraverso il proxy, digitiamo:
```sh
curl -v -x socks5://pi:raspberry@127.0.0.1:1080 http://www.google.com/
```
Un risultato come segue vuol dire: "il proxy lavora agregiamente!"
```sh
*   Trying 127.0.0.1...
* TCP_NODELAY set
* SOCKS5 communication to www.google.com:80
* SOCKS5 connect to IPv4 ***.***.***.*** (locally resolved)
* SOCKS5 request granted.
* Connected to (nil) (127.0.0.1) port 1080 (#0)
> GET / HTTP/1.1
> Host: www.google.com
> User-Agent: curl/7.52.1
> Accept: */*
>
< HTTP/1.1 302 Moved Temporarily
< Cache-Control: private
< Content-Type: text/html; charset=UTF-8
< Referrer-Policy: no-referrer
< Location: http://www.google.it/?gfe_rd=cr&dcr=0&ei=5GnbWYXJM8nA8gfBl5zwBg
< Content-Length: 268
< Date: Mon, 09 Oct 2017 12:21:56 GMT
< X-Cache: MISS from [###]
< X-Cache-Lookup: MISS from [###]n:3128
< Via: 1.1 [###] (squid/3.3.13)
< Connection: keep-alive
<
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.it/?gfe_rd=cr&amp;dcr=0&amp;ei=5GnbWYXJM8nA8gfBl5zwBg">here</A>.
</BODY></HTML>
* Curl_http_done: called premature == 0
* Connection #0 to host www.google.com left intact
```
Aggiungiamo ora un utente per il nostro proxy, in quanto uscire con le password di amministrazione non è mai bello! Digitiamo:
```sh
adduser proxymatteo
```
usiamo solo lettere minuscole per il nome, e seguiamo le domande che ci vengono poste:
```sh
Adding user `proxymatteo' ...
Adding new group `proxymatteo' (1002) ...
Adding new user `proxymatteo' (1002) with group `proxymatteo' ...
Creating home directory `/home/proxygiuseppe' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:    <--- SCEGLIAMO E DIGITIAMO LA NOSTRA PASSWORD
Retype new UNIX password:    <--- DIGITIAMO NUOVAMENTE LA PASSWORD
passwd: password updated successfully
Changing the user information for proxymatteo
Enter the new value, or press ENTER for the default
        Full Name []:     <--- DIAMO INVIO SENZA SCRIVERE NULLA
        Room Number []:     <--- DIAMO INVIO SENZA SCRIVERE NULLA
        Work Phone []:     <--- DIAMO INVIO SENZA SCRIVERE NULLA
        Home Phone []:    <--- DIAMO INVIO SENZA SCRIVERE NULLA
        Other []:     <--- DIAMO INVIO SENZA SCRIVERE NULLA
Is the information correct? [Y/n] Y     <--- Y e INVIO PER CONFERMARE
```
# Ricapitolando #2
- Abbiamo installato un terminale per la Shell (SSH)
- Abbiamo installato il server Proxy (Dante)
- Abbiamo testato il funzionamento
- Abbiamo creato l'utente con il quale collegarci.

Nei prossimi step, capiremo come rendere il nostro proxy raggiungibile dalla rete esterna! ...e qui servirà un po' di auto gestione :)

# Gestione del router
In questi ultimi step, dovremo configurare il nostro router per accettare connessioni verso il nostro proxy. Come sapete (e se non lo sapete è il momento di impararlo), il nostro router agisce da "muro" con la rete esterna. Permette a noi dall'interno di navigare verso l'esterno, ma non accetta che l'esterno possa entrare nella nostra rete, se non opportunamente configurato. Noi dovremo dire al nostro router di accettare le connessioni che arrivano sulla porta 1080 e rigirarle verso il nostro Raspberry, così da essere interpretate dal proxy.

Apriamo nuovamente FING, e cerchiamo nella lista degli oggetti in rete, due freccie che guardando verso destra si incrociano. Quello sarà il nostro router, e potrebbe avere indirizzi tipo 192.168.1.1, 192.168.1.254, 192.168.0.1, 192.168.0.254.
Apriamo una finestra del browser internet, e digitiamo l'indirizzo, ci troveremo davanti una finestra di login, per poter gestire il nostro modem.

I router vengono protetti da password di amministrazione, che generalmente (se non cambiate privatamente) sono quelle di default. Ora vi allego le password standard per alcuni modelli, ma potrebbe essere richiesta qualche ricerca per trovare la vostra specifica. In caso di non funzionamento, cercate in rete il modello del vostro router, seguito da "default password", esempio "TPLINK wa850re default password".
```sh
TPLINK
user: admin
password: admin

NETGEAR
user: admin
password1: password (per i piu recenti)
password2: 1234 (per i meno recenti)

HUAWEI
user: user
password: user

FASTWEB
user1: Administrator (rete in VOIP)
user2: fastweb (telefoni connessi ancora al filtro)
password: <vuoto>

MODEM TIM
user: Administrator (dovrebbe essere preimpostato)
password: <vuoto>

MODEM VODAFONE
user: vodafone 
password: (dovrebbe essere preimpostato)

```

Siamo dentro, nel cervellone della nostra rete domestica, aguzziamo la vista e andiamo alla ricerca di una voce chiamata: PORT FORWARDING, solitamente lo troviamo sotto NAT/Advanced Settings. Al suo interno potremo inserire una serie di "regole" per instradare il nostro traffico esterno. 

Dovremo procedere in questo modo, ovvero quello di inviare tutto il traffico (se richiesto: sia TCP che UDP) verso la porta 1080 dell'ip del nostro Raspberry. Esempi
```sh
Rule Index : 1
Application : Proxy Raspberry
Protocol : ALL
Start Port Number : 1080
End Port Number : 1080
Local IP Address : 192.168.1.104 <- IP del vostro Raspberry
```
oppure
```sh
Service Name    Proxy Raspberry
Service Type	TCP/UDP
External Starting Port  1080
External Ending Port	1080
Use the same port range for Internal port <- YES
Internal IP Address	192 . 168 . 0 . 104 <- IP del vostro Raspberry
```
Creata la regola, salviamo e applichiamo le modifiche. 

Un altro step da effettuare, è quello di rendere il nostro IP statico. Se siamo clienti Fibra, gli IP sono solitamente statici, altrimenti, se siamo in ADSL potrebbe essere necessario uno step aggiuntivo. Cerchiamo all'interno del nostro router la voce DDNS (o anche Dynamic DNS). Seguiamo le istruzioni che il nostro router ci da, e portiamole a termine.

Questa operazione è necessaria perchè, per via dell'indirizzo IP di casa dinamico, saremo soggetti ad una variazione (su base bi-giornaliera o settimanale). Se inseriamo il nostro IP nel BOT di Instagram e successivamente questo cambia, il nostro Bot interromperà le azioni. Se invece siamo sicuri di avere un indirizzo IP statico (vedi Fastweb o Vodafone), potremmo tranquillamente inserire l'ip nelle configurazioni del proxy.

# Configuriamo il BOT
Prendiamo il nostro Bot che stiamo utilizzando, e cerchiamo nelle impostazioni, l'area relativa al server Proxy
Compiliamola con i nostri dati!
```sh
Use proxy for this account ? <- Acceso!
Indirizzo proxy: 213.82.xxx.xxx
Porta proxy: 1080
Username: proxymatteo
Password: passwordproxy
```
Prima di dare il salvataggio, verifichiamo che le impostazioni siano salvate correttamente con un test del proxy, solitamente vicino alla configurazione.

Cari amici. Abbiamo finito! Godetevi il vostro Proxy privato e senza canoni! :) 
