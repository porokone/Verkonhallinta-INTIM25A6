# Viikko 2 - SNMP ja verkon perustason valvonta (Ver 0.3)

## 1. Johdanto

### Mikä on SNMP?

IBM:n määrittelyn mukaan SNMP on avoin arkkitehtuuri jonka avulla verkon laitteet voivat välittää valvontatietoja verkossa. Verkossa on hallintalaitteita jotka valvovat laitteiden välittämiä MIB resursseja/elementtejä ja niiden perusteella voidaan valvoa esimerkiksi verkkoporttien käyttöastetta, servereiden kuormitusta yms. jolloin tuleviin ongelmiin voidaan reagoida jo ennen kuin ne muodustuu ongelmiksi.

---
## 2. Asennus

### Miten SNMP-agentti asennettiin?

Jokaiseen kohteeseen (web1, db1 ja branch-client) asennettiin snmp-agentti seuraavilla komennoilla. Samalla asennettiin tarvittavia työkaluja ja nano-editori.

Snmp-agentin asennus
```bash
apt update && apt install snmp snmpd -y
```

IP työkalujen ja nano-editorin asennus
```bash
apt install nano net-tools iputils-ping -y
```

Seuraavaksi pitää tarkistaa kontin ip-osoite, joka lisätään snmp:n ip-listaan. Tässä harjoituksessa käytin Management LANin verkkoa 172.20.20.0/24 joka löytyi konteista eth0:sta.

```bash
ip a
```

Seuraavaksi muutettiin tai lisättiin seuraavat kohdat etc/snmp/ hakemistossa sijaitsevaan snmpd.conf

```bash
nano etc/snmp/snmpd.conf
```

Sieltä etsin seuraavan kohdan

```text
SECTION: Access Control Setup
```
Jonka alle lisäsin seuraavan
```text
view   systemonly  included   .1.3.6.1.2
```



---
## 3. Kerätyt tiedot
Kuvaukset ja tulosteet.

---
## 4. Verkkorajapinnat
SNMP:n avulla kerätyt rajapintatiedot.

---
## 5. OID-analyysi
OID-objektien käyttötarkoitus.

---
## 6. Pohdinta
Omat havainnot SNMP:n hyödyistä ja rajoituksista.

---