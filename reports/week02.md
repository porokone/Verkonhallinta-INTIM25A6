# Viikko 2 - SNMP ja verkon perustason valvonta (Ver 1.0)

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

Sitten lisätään ip-osoite agentin osoitteeksi. Eli etsin seuraavan kohdan.

```text
agentaddress  127.0.0.1,[::1]
```

Sen perään lisäsin kyseisen kontin 172. - verkon osoitteen. Tässä tapauksessa käytän esimerkkinä branch-clientin osoitetta.

```text
agentaddress  127.0.0.1,[::1],172.20.20.8
```

Tallennetaan ja poistutaan nano editorista,
Ctrl + s (tallentaa) ja Ctrl + x (poistuu)

Seuraavaksi muokataan samassa hakemistossa olevaa snmp.conf

```bash
nano etc/snmp/snmp.conf
```

Etsitään sieltä seuraava kohta

```text
mibs :
```

Muutetaan se muotoon

```text
# mibs :
```

tallennetaan ja poistutaan.

Sitten vielä ladataan snmp-mibs-downloader

```bash
apt install snmp-mibs-downloader
```

Seuraavaksi käynnistetään uudestaan snmpd-palvelu.

```bash
service snmpd restart
```

Sen jälkeen seuraavilla komennoilla voidaan tarkistaa onko palvelu päällä

```bash
service snmpd status
```

ja mitä osoitetta se käyttää

```bash
ss -lunp | grep 161
```
pitäisi tulla esimerkiksi seuraavanlaisia osoitteita tulosteeksi
```text
UNCONN 0      0        172.20.20.8:161        0.0.0.0:*    users:(("snmpd",pid=4782,fd=9))
UNCONN 0      0          127.0.0.1:161        0.0.0.0:*    users:(("snmpd",pid=4782,fd=7))
UNCONN 0      0              [::1]:161           [::]:*    users:(("snmpd",pid=4782,fd=8))
```

### Snmp:n asennus ansible-konttiin

Asennetaan snmp ja samalla tavalla kuin asennettiin valvottaviin kohteisiinkin, asennetaan snmp-mibs-downloader myös ja muokataan snmp.conf samalla tavalla kuin yllä on kerrottu.

Sitten voidaan kokeilla hakea tietoja suoraan agenteilta.

```bash
snmpget -v2c -c public web1 sysName.0
snmpget -v2c -c public db1 sysName.0
snmpget -v2c -c public branch-client sysName.0
```

ja tulosteiksi pitäisi tulla

```text
SNMPv2-MIB::sysName.0 = STRING: web1
SNMPv2-MIB::sysName.0 = STRING: db1
SNMPv2-MIB::sysName.0 = STRING: branch-client
```

---
## 3. Kerätyt tiedot

Tietojen keräykseen käytettiin seuraavia komentoja (muokkaa kontin nimeä tarvittaessa)

```bash
snmpget -v2c -c public web1 sysName.0
snmpget -v2c -c public web1 sysDescr.0
snmpget -v2c -c public web1 sysUpTime.0
```

| Laite | Nimi | Käyttäjärjestelmä | Uptime |
|---|---|---|---|
| web1 | web1 | Linux web1 6.18.33.2-microsoft-standard-WSL2 | 4:17:29.09 |
| db1 | db1 | Linux db1 6.18.33.2-microsoft-standard-WSL2 | 2:22:43.17 |
| branch-client | branch-client | Linux branch-client 6.18.33.2-microsoft-standard-WSL2 | 2:11:48.37 |

---
## 4. Verkkorajapinnat

Seuraavilla komennoilla selviteltiin kontin rajapinnat sekä siihen liittyvät ip-osoitteet.

```bash
snmpwalk -v2c -c public web1 ifDescr
snmpwalk -v2c -c public web1 ipAdEntIfIndex
```

Ensimmäinen antaa tulosteeksi rajapinnat ja toinen ip-osoitteet. Ne yhdistelemällä voidaan muodostaa seuraava taulu.

| Nimi | Rajapinta | IP-osoite |
|---|---|---|
| Web1 | lo | 127.0.0.1 |
| Web1 | eth0 | 172.20.20.10 |
| Web1 | eth1 | 10.10.20.101 |
| Db1 | lo | 127.0.0.1 |
| Db1 | eth0 | 172.20.20.3 |
| Db1 | eth1 | 10.10.20.102 |
| Branch-client | lo | 127.0.0.1 |
| Branch-client | eth0 | 172.20.20.8 |
| Branch-client | eth1 | 10.10.30.101 |

---
## 5. OID-analyysi

| OID | Tarkoitus |
|---|---|
| sysName.0 | Laitteen nimi |
| sysDescr.0 | Järjestelmän kuvaus |
| sysUpTime.0 | Aika, kauan järjestelmä on ollut käynnissä |
| ifDescr | Antaa rajapintojen nimet |
| ifOperStatus | Näyttää rajapintojen tilan |

---
## 6. Pohdinta

### 1. Mitä hyötyä SNMP:stä on verkonhallinnassa?

SNMP:n avulla voi kerätä jokaisen kriittisen laitteen valvonnan kannalta tärkeitä tietoja keskitetysti ja havaita esimerkiksi suuria liikennemääriä tai rajapintojen putoamisia jolloin voidaan keskittää verkon parannukset oikeasti sinne missä niitä tarvitaan eikä tarvitse arpoa missä ne ongelmat ovat.

### 2. Mitä tietoa SNMP:n avulla voidaan kerätä?

Ihan laitteen nimestä rajapintojen tilojen seuraamiseen ja rajapintojen liikenteen määrästä eri komponenttien käyttöasteeseen. Riippuu paljon käytettävästä laitteesta.

### 3. Mitä ongelmia yhteisöpohjaisessa SNMPv2:ssa on?

Isoin ongelma on että se ei tarjoa salausta tai käyttäjäkohtaista tunnistautumista vaan lähettää tiedot salaamattomana verkossa.

### 4. Missä tilanteissa käyttäisit mieluummin SNMPv3:a?

Tuotantoverkossa ja tietoturvan kannalta tärkeissä ympäristöissä.

### Omaa pohdintaa

Isoin kysymysmerkki tuli tosiaan tietoturvan kannalta SNMPv2:sen ja SNMPv3:sen suhteen.
SNMPv2 kuitenkin tarjoaa ison tietoturvariskin verkkoon jos hyökkääjä pääsee siihen käsiksi koska sitä kautta voi kartoittaa käytännössä koko verkon mihin ei välttämättä muuten olisi mahdollisuutta.

---