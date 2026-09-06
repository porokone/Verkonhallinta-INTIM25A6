# Viikko 2 - SNMP ja verkon perustason valvonta (Ver 0.6)

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

| Kontin nimi | Kuvaus | Käyttöaika |
|---|---|---|
| web1 | Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 4:17:29.09 |
| db1 | Linux db1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 2:22:43.17 |
| branch-client | Linux branch-client 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 2:11:48.37 |

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