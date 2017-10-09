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
