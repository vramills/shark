# Pràctica: Analitzador de protocols Wireshark

## Configuracions inicials

Per fer aquesta pràctica necessitem el Kali Linux amb Wireshark instal·lat i la targeta de xarxa configurada en mode pont amb les següents dades:

- `IP:` 192.168.4.26/24
- `Porta d'enllaç:` 192.168.4.254
- `DNS:` 8.8.8.8

Accedim a la configuració de xarxa i configurem l'adaptador manualment:

<img src="img/1.png">

---

## 1. Anàlisi en viu

### 1.1 ICMP

Iniciem la captura a Wireshark sobre la targeta `eth0` i executem un ping al gateway des d'un terminal:

```bash
ping 192.168.4.10
```

<img src="img/2.png">

Aturem la captura i apliquem el filtre `icmp`. Podem veure els paquets Echo request i Echo reply:

<img src="img/3.png">

Cliquem sobre el paquet `Echo request` i despleguem la secció `Internet Control Message Protocol` al panell inferior. Podem veure que el tipus és `8`:

<img src="img/4.png">

Cliquem sobre el paquet `Echo reply` i veiem que el tipus és `0`:

<img src="img/5.png">

### Pregunta

1. Quin número de tipus de ICMP té la petició d'eco i quin la resposta d'eco? Com ho veus?

El tipus ICMP de la petició d'eco (Echo request) és el `tipus 8` i el de la resposta d'eco (Echo reply) és el `tipus 0`.

---

### 1.2 Mode promiscu

Activem el mode promiscu a la configuració de la targeta de xarxa del VirtualBox seleccionant `"Permet-ho tot"`:

<img src="img/6.png">

### Pregunta

1. Fes una captura de trànsit, mentre navegues des de la màquina física. Quin trànsit pots veure relacionat amb el teu PC?

Iniciem una nova captura i naveguem des del PC físic. Podem veure tràfic de tota la xarxa, incloent paquets DNS de la navegació des del PC físic, pel que podem veure l'accés que s'ha fet a la `Wikipedia`:

<img src="img/7.png">

---

### 1.3 DNS

Apliquem el filtre `dns and ip.addr == 192.168.4.26` i executem una consulta DNS:

```bash
nslookup www.xtec.cat
```

<img src="img/8.png">

Podem veure els dos paquets DNS: la petició (query) i la resposta (response):

<img src="img/9.png">

Cliquem sobre el paquet `response` i despleguem la secció `Answers`. Veiem que l'adreça IP de `www.xtec.cat` és `83.247.151.214`, que coincideix amb la del `nslookup`:

<img src="img/10.png">

---

### 1.4 ARP

Apliquem el filtre `arp` i executem un ping al gateway per generar tràfic ARP.

```bash
ping -c 1 192.168.4.254
```

Podem veure la petició (Who has 192.168.4.254?) i la resposta:

<img src="img/11.png">

### Pregunta

1. Quina adreça MAC té el gateway de la xarxa? Quin és el fabricant de la seva NIC?

La MAC del gateway (192.168.4.254) és `08:00:27:be:1b:a8`. El fabricant és `PCS Systemtechnik GmbH` (VirtualBox), tal com es pot veure a la columna Source amb el prefix `PCSSystemtec`.

---

## 2. Anàlisi de captures. Arxius

Descarreguem els fitxers `captura1.pcapng` i `captura2.pcapng` del repositori i els obrim des de `Archivo → Abrir`:

<img src="img/12.png">

<img src="img/13.png">

---

### 2.1 ARP — captura1.pcapng

Apliquem el filtre `arp and arp.src.proto_ipv4 == 192.168.1.1` per veure els paquets ARP de l'equip amb IP `192.168.1.1`:

<img src="img/14.png">

### Pregunta

1. Pots saber quina adreça MAC té l'equip amb adreça 192.168.1.1?

La MAC de l'equip amb IP 192.168.1.1 és `d4:76:ea:0f:fd:58`.

---

### 2.2 FTP — captura1.pcapng

Apliquem el filtre `ftp` per veure la sessió FTP:

<img src="img/15.png">

### Pregunta

1. Quin és el password de l'usuari que inicia sessió? Quin nom té el fitxer que es descarrega del servidor?

- `Usuari:` anonymous
- `Contrasenya:` contra (visible en text clar al paquet `PASS contra`)
- `Fitxer descarregat:` README.txt

---

### 2.3 Telnet — captura1.pcapng

Apliquem el filtre `telnet`:

<img src="img/16.png">

I fem clic dret sobre un paquet → `Seguir → Flujo TCP`:

<img src="img/17.png">

Podem veure el contingut de la sessió en text clar, on es veu que s'ha iniciat una sessió Telnet a `towel.blinkenlights.nl` i s'està visualitzant una animació ASCII de Star Wars:

<img src="img/18.png">

### Pregunta

1. Pots veure el que veia l'usuari en connectar al telnet? Explica què és. Quins caràcters composen la nau espacial petita (posar com a resposta)?

En connectar al telnet es veu que s'ha iniciat una sessió a `towel.blinkenlights.nl` i s'està visualitzant una `animació ASCII de Star Wars` on s'observa una `Nau espacial petita`, composada pels següents caràcters:

```
O=<88>=O<E
```

2. A quin domini pertany l'adreça on ens connectem?

El Domini de la IP destí (94.142.241.111), pertany a `towel.blinkenlights.nl`.

---

### 2.4 SSH — captura1.pcapng

Apliquem el filtre `ssh and frame.len == 326` per trobar el paquet de 326 bytes:

<img src="img/19.png">

Cliquem sobre el paquet i veiem les dades xifrades al panell inferior:

<img src="img/21.png">

<img src="img/20.png">

### Pregunta

1. Pots saber a quin domini pertany l'adreça del servidor?

No podem saber al domini que pertany l'adreça del servidor SSH, ja que els paquets SSH estan xifrats i no contenen informació de domini en text clar. Només podem veure les adreces IP dels servidors SSH, que són `205.166.94.15` i `205.166.94.17`.

2. Pots veure el contingut de les dades de la sessió? Enganxa les dades que conté el paquet ssh de longitud total 326 bytes.

No, SSH xifra totes les comunicacions. Tots els paquets apareixen com `Encrypted packet` i les dades al panell inferior són incomprensibles sense la clau de desxifrat. 

Pel que les dades del paquet de `326 bytes` estan completament xifrades.

---

### 2.5 Correu electrònic — captura2.pcapng

Obrim `captura2.pcapng` i apliquem el filtre `smtp`:

<img src="img/22.png">

<img src="img/23.png">

Fem clic dret sobre el paquet que conté el missatge → `Seguir → Flujo TCP`:

<img src="img/24.png">

Podem veure el contingut complet del correu en text clar:

<img src="img/25.png">

### Pregunta

S'ha enviat el següent correu:

- `Remitent:` `pau`
- `Destinatari:` `root`
- `Contingut del missatge:` `mensaje ultrasecreto para el administrador`
- `Servidor SMTP:` `ubuntu-server` (Postfix)