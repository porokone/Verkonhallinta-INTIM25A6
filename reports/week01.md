# Viikko 1 – Verkon dokumentointi (Ver 1.0)

## 1. Johdanto

Tässä harjoituksessa tutustuttiin Containerlabilla ja Docker-konteilla toteutettuun virtuaaliseen verkkoympäristöön. Ympäristö oli kopioitu annetusta Github-reposta jonka pystytin Windows 11 - koneelle WSL2.0 - avulla.

Tehtävässä jouduin käyttämään useita, itselleni uusia komentoja joiden avulla selvitin verkkoa käyttäviä laitteita. Aloitin scannailemalla verkkoa client1:stä ilman apua valmiista topologiasta. Lopuksi täydensin omat löydökseni valmista topologiaa apuna käyttäen.

---

## 2. Verkkokaavio

Verkon rakennetta tutkin reposta löytyvän verkkotyökalu-oppaassa (topology.md) kerrottujen komentojen antamien tulosteiden perusteella.

```mermaid
flowchart LR

    subgraph USER["User LAN - 10.10.10.0/24"]
        CLIENT1["client1<br/>10.10.10.101/24"]
        R1["r1<br/>10.10.10.1"]
    end

    subgraph SERVER["Server LAN - 10.10.20.0/24"]
        SERVER1["laite<br/>10.10.20.101"]
        SERVER2["laite<br/>10.10.20.102"]
    end

    subgraph MGMT["Management LAN - 10.10.99.0/24"]
        MGMTLAITE1["10.10.99.1"]
    end

    subgraph BRANCH["Branch Office - 10.10.30.0/24"]
        R3["r3<br/>10.10.30.1"]
        BRANCHCLIENT["branch-client<br/>10.10.30.101"]
    end

    R1 <-->|"10.255.12.0/30"| R2["r2"]
    R2 <-->|"10.255.23.0/30"| R3["r3"]

    R2 --- SERVER
    R2 --- MGMT
```

Verkon topologia kuvattuna Containerlabin topologiasta.

```mermaid
flowchart LR

    subgraph USER["User LAN - 10.10.10.0/24"]
        client1["Client1</br>10.10.10.101"]
        attacker["Attacker</br>10.10.10.200"]
    end
    r1["R1</br>10.10.10.1"]
    r1r2["R1-R2 LAN</br>10.255.12.1 - 10.255.12.2"]
    r2["R2</br>10.10.20.1"]
    r2r3["R2-R3 LAN</br>10.255.23.1 - 10.255.23.2"]
    r3["R3</br>10.10.30.1"]
    branch["Branch-client</br>10.10.30.101"]

    srv-bp["Srv-bp"]
    mgmt-bp["Mgmt-bp"]

    srv["Server LAN"]

    subgraph SRVLAN["Server LAN 10.10.20.0/24"]
        web1["Web1</br>10.10.20.101"]
        db1["Db1</br>10.10.20.102"]
    end

    mgmt["Management LAN"]

    subgraph MGMT["Management LAN 10.10.99.0/24</br>"]
        cadvisor["Cadvisor"]
        prometheus["Prometheus"]
        ansible["Ansible"]
        grafana["Grafana"]
        syslog["Syslog"]
        zabbix["Zabbix"]
    end

    client1 --- r1
    attacker --- r1

    r1 --- r1r2
    r1r2 --- r2
    r2 --- r2r3
    r2r3 --- r3
    r3 --- branch

    r2 --- srv
    srv --- web1
    srv --- db1

    r2 --- mgmt
    mgmt --- cadvisor
    mgmt --- prometheus
    mgmt --- ansible
    mgmt --- grafana
    mgmt --- syslog
    mgmt --- zabbix
```

---

## 3. Laiteluettelo

Laiteluettelo scannailun perusteella

| IP-osoite | Laite | Avoimet portit | Tarkoitus |
|---|---|---|---|
| 10.10.10.1 | r1 | 2601,2604 |  |
| 10.10.10.101 | client1 | 22, 9100 | Käytettävä kontti |
| 10.10.10.254 | r1 | 2601,2604 | Sama MAC-osoite |
| 10.10.20.1 | r2 | 2601,2604 | Reititin |
| 10.10.20.101 | web1/db1 | 22,9100 | Selkeää varmuutta en löytänyt (myöhemmin tarkistin suoraan kontilta ip:n) |
| 10.10.20.102 | web1/db1 | 22,9100 | Selkeää varmuutta en löytänyt (myöhemmin tarkistin suoraan kontilta ip:n) |
| 10.10.30.1 | r3 | 2601,2604 | Reititin |
| 10.10.30.101 | branch-client | 22 | Branch Officen kontti |
| 10.10.99.1 |   | 2601,2604 |   |
| 10.255.12.1 | r1 | 2601,2604 | Yhdistelemällä useita tuloksia näyttäisi olevan r1-r2 välinen verkko |
| 10.255.12.2 | r2 | 2601,2604 | Yhdistelemällä useita tuloksia näyttäisi olevan r1-r2 välinen verkko |
| 10.255.23.1 | r2 | 2601,2604 | Yhdistelemällä useita tuloksia näyttäisi olevan r2-r3 välinen verkko |
| 10.255.23.2 | r3 | 2601,2604 | Yhdistelemällä useita tuloksia näyttäisi olevan r2-r3 välinen verkko |

Laiteluettelo containerlabin topologian avulla

| Laite | Tarkoitus |
|---|---|
| r1 | User LAN -verkon reititin |
| r2 | Yhdistää r1, r3 ja pari kytkintä joiden perässä on valvonta- ja serverikontit |
| r3 | Branch Office -verkon reititin |
| client1 | User LAN -verkon asiakaskontti |
| attacker | Attacker, käytetään ilmeisesti jossain vaiheessa kun ei näy arp-taulukossa ja näyttäisi olevan portit kiinni |
| web1 | Web-palvelin |
| db1 | Tietokantapalvelin |
| branch-client | Branch Officen kontti |
| ansible | Automaation hallintakontti |
| prometheus | Mittaus- ja monitorointitiedon keräämiseen tarkoitettu kontti |
| grafana | Prometheuksen datan graaffiseen näyttöön tarkoitettu kontti |
| zabbix | Verkon ja palveluiden valvontaan tarkoitettu kontti |

### Client1

Client1 on User LAN -verkossa sijaitseva kontti. Kontilla on kaksi verkkoa:

| Liitäntä | IP-osoite | Verkko |
|---|---|---|
| eth0 | 172.20.20.5/24 | Containerlab/Docker-hallintaverkko |
| eth1 | 10.10.10.101/24 | User LAN |

Client1:n oletusyhdyskäytävä on `10.10.10.1` joten muihin harjoitusympäristön verkkoihin suuntautuva liikenne kulkee eth1-liitännän kautta.

---

## 4. IP-suunnitelma

Ympäristössä havaittiin seuraavat verkot:

| Verkko | Tarkoitus | Yhdyskäytävä |
|---|---|---|
| 10.10.10.0/24 | User LAN | 10.10.10.1 |
| 10.10.20.0/24 | Server LAN | 10.10.20.1 |
| 10.10.30.0/24 | Branch Office | 10.10.30.1 |
| 10.10.99.0/24 | Management LAN | 10.10.99.1 |
| 10.255.12.0/30 | r1-r2 välinen yhteys | - |
| 10.255.23.0/30 | r2-r3 välinen yhteys | - |

### Havaitut IP-osoitteet

#### User LAN – 10.10.10.0/24

| IP-osoite | Laite / havainto |
|---|---|
| 10.10.10.1 | Oletusyhdyskäytävä / r1 |
| 10.10.10.101 | client1 |
| 10.10.10.254 | Vastasi pingiin, sama MAC-osoite kuin 10.10.10.1 |

#### Server LAN – 10.10.20.0/24

| IP-osoite | Laite / havainto |
|---|---|
| 10.10.20.1 | Yhdyskäytävä |
| 10.10.20.101 | Web1 |
| 10.10.20.102 | Db1 |

#### Branch Office – 10.10.30.0/24

| IP-osoite | Laite / havainto |
|---|---|
| 10.10.30.1 | Yhdyskäytävä |
| 10.10.30.101 | branch-client |

#### Management LAN – 10.10.99.0/24

| IP-osoite | Laite / havainto |
|---|---|
| 10.10.99.1 | Yhdyskäytävä |

#### Reitittimien välinen verkko – 10.255.12.0/30

| IP-osoite | Laite / havainto |
|---|---|
| 10.255.12.1 | Reitittimen liitäntä |
| 10.255.12.2 | Reitittimen liitäntä |

#### Reitittimien välinen verkko – 10.255.23.0/30

| IP-osoite | Laite / havainto |
|---|---|
| 10.255.23.1 | Reitittimen liitäntä |
| 10.255.23.2 | Reitittimen liitäntä |

### Containerlabin hallintaverkko

Varsinaisen harjoitusverkon lisäksi konteilla on erillinen `172.20.20.0/24`-verkko. Esimerkiksi client1 käyttää siinä osoitetta `172.20.20.2/24`.

Tämä verkko on erotettu varsinaisista 10.x.x.x-harjoitusverkoista ja sitä käytetään Containerlab/Docker-ympäristön konttien hallintaan.

IP-osoitteet vaihtuivat kun jouduin resetoimaan kontti-ympäristön. Osoitteisto päivitetty vastaamaan nykyistä tilannetta.


| IP-osoite | Avoimet portit | Havainnot |
|---|---|---|
| 172.20.20.1 | 3000,8000,8080,9090 | Tämä viittaa kontti-hostiin huomioiden ympäristön |
| 172.20.20.2 | 22,9100 | Client1 |
| 172.20.20.3 | 3000 | Grafana |
| 172.20.20.4 | 9090 | Prometheus |
| 172.20.20.5 |   | Kaikki portit kiinni, Srv-bp |
| 172.20.20.6 | 22 | Attacker |
| 172.20.20.7 | 2601,2604 | Reititin 3 |
| 172.20.20.8 | 22,9100 | Web1 |
| 172.20.20.9 | 22,9100 | Db1 |
| 172.20.20.10 | 2601,2604 | Reititin 1 |
| 172.20.20.11 | 22 | Ansible |
| 172.20.20.12 |   | Kaikki portit kiinni, Mgmt-bp |
| 172.20.20.13 | 8080 | Cadvisor |
| 172.20.20.14 | 22 | Branch client |
| 172.20.20.15 | 2601,2604 | Reititin 2 |
| 172.20.20.16 | 80 | Zabbix |
| 172.20.20.17 | 8080 | *** Ei löytynyt listalta *** |
| 172.20.20.50 | 514 | Syslog |

---

## 5. Reitityksen analyysi

### Client1:n verkkoliitännät

Client1:n verkkoliitännät tutkittiin komennolla:

```bash
ip a
```

Tulosteesta havaittiin kaksi verkkoliitäntää:

```text
eth0: 172.20.20.2/24
eth1: 10.10.10.101/24
```

### Client1:n reititystaulu

Reititystaulu tarkistettiin komennolla:

```bash
ip route
```

Tuloste:

```text
default via 10.10.10.1 dev eth1
10.10.10.0/24 dev eth1 proto kernel scope link src 10.10.10.101
172.20.20.0/24 dev eth0 proto kernel scope link src 172.20.20.2
```

Reititystaulun perusteella client1:n oletusyhdyskäytävä on `10.10.10.1`. User LAN on suoraan saavutettavissa eth1-liitännän kautta ja Containerlabin hallintaverkko eth0-liitännän kautta.

### Yhteys Server LAN -verkkoon

Yhteyttä Server LAN -verkossa olevaan osoitteeseen `10.10.20.101` testattiin komennolla:

```bash
ping -c 4 10.10.20.101
```

**Tuloste:**

```text
root@client1:/# ping -c 4 10.10.20.101
PING 10.10.20.101 (10.10.20.101) 56(84) bytes of data.
64 bytes from 10.10.20.101: icmp_seq=1 ttl=62 time=0.086 ms
64 bytes from 10.10.20.101: icmp_seq=2 ttl=62 time=0.074 ms
64 bytes from 10.10.20.101: icmp_seq=3 ttl=62 time=0.072 ms
64 bytes from 10.10.20.101: icmp_seq=4 ttl=62 time=0.070 ms

--- 10.10.20.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3131ms
rtt min/avg/max/mdev = 0.070/0.075/0.086/0.006 ms
```

Tracerouten perusteella liikenne kulkee seuraavaa reittiä:

```text
root@client1:/# traceroute 10.10.20.101
traceroute to 10.10.20.101 (10.10.20.101), 30 hops max, 60 byte packets
 1  10.10.10.1 (10.10.10.1)  0.507 ms  0.436 ms  0.416 ms
 2  10.255.12.2 (10.255.12.2)  0.403 ms  0.375 ms  0.360 ms
 3  10.10.20.101 (10.10.20.101)  0.344 ms  0.230 ms  0.202 ms
```

### Yhteys Branch Office -verkkoon

Yhteyttä branch-clientiin testattiin komennolla:

```bash
ping -c 4 10.10.30.101
```

**Tuloste:**

```text
root@client1:/# ping -c 4 10.10.30.101
PING 10.10.30.101 (10.10.30.101) 56(84) bytes of data.
64 bytes from 10.10.30.101: icmp_seq=1 ttl=61 time=0.089 ms
64 bytes from 10.10.30.101: icmp_seq=2 ttl=61 time=0.071 ms
64 bytes from 10.10.30.101: icmp_seq=3 ttl=61 time=0.068 ms
64 bytes from 10.10.30.101: icmp_seq=4 ttl=61 time=0.071 ms

--- 10.10.30.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3043ms
rtt min/avg/max/mdev = 0.068/0.074/0.089/0.008 ms
```

Reitti tutkittiin komennolla:

```bash
traceroute 10.10.30.101
```

Havaittu reitti:

```text
root@client1:/# traceroute 10.10.30.101
traceroute to 10.10.30.101 (10.10.30.101), 30 hops max, 60 byte packets
 1  10.10.10.1 (10.10.10.1)  0.563 ms  0.514 ms  0.501 ms
 2  10.255.12.2 (10.255.12.2)  0.488 ms  0.462 ms  0.446 ms
 3  10.255.23.2 (10.255.23.2)  0.432 ms  0.328 ms  0.274 ms
 4  10.10.30.101 (10.10.30.101)  0.249 ms  0.173 ms  0.145 ms
```

Tuloksen perusteella liikenne kulkee User LAN -verkosta usean reitittimen kautta Branch Office -verkkoon.

### Verkkojen kartoitus Nmapilla

Verkkojen aktiivisia laitteita kartoitettiin esimerkiksi komennoilla:

```bash
nmap --traceroute -sn 10.10.20.0/24
```

Vastaavalla tavalla tutkittiin verkot:

```text
10.10.10.0/24
10.10.20.0/24
10.10.30.0/24
10.10.99.0/24
10.255.12.0/30
10.255.23.0/30
```

Nmapin avulla pystyttiin selvittämään aktiivisia IP-osoitteita ja avoimia portteja sekä traceroutea tarkastelemaan liikenteen kulkemaa reittiä eri verkkoihin. Arp-tauluja käytin tutkiessa 10.10.10.0/24 verkkoa.

Listaan alle käyttämiäni komentoja kun yritin jokaista verkkoa scannailla ja löytää vastaavia laitteita. Tulosteita en liitä tähän koska ne usein sisälsi samoja tietoja kuin yllä on lueteltu ja osa oli niin raskaita ajaa kun kokeilin, varsinkin nmapin eri parametrejä että ne usein ei edes tulostanut mitään tai tuloste oli pitkä.

```text
nmap -sn 10.10.10.0/24 (etsii vain aktiiviset hostit, ei porttiscannausta)
nmap -PR 10.10.10.0/24 (Tekee arp-kyselyn ja scannaa portit)
nmap -sV 10.10.10.0/24 (Yrittää tunnistaa avoimissa porteissa vastaavat palvelut)
nmap -Pn 10.10.10.0/24 (Olettaa hostin aktiiviseksi ja tekee portti-scannauksen)
```

---

## 6. Yhteenveto

Harjoituksessa kartoitettiin Containerlabilla toteutetun virtuaalisen verkon rakennetta. Verkon tutkimisessa käytettiin Linuxin verkkotyökaluja, joiden avulla selvitettiin laitteiden IP-osoitteita, verkkojen välisiä yhteyksiä sekä liikenteen käyttämiä reittejä.

Nimeämisessä käytin apuna tehtävänannossa annettuja nimiä.

### Mikä vei eniten aikaa?

Suurin osa ajasta meni siihen että sain ympäristön toimimaan, linkitettyä Githubiin ja tutustuessa Mermaidiin (valitsin Mermaidin sen takia että tämä koko kurssin palautukset on tarkoitus tehdä Githubiin ja Mermaidin integraatio Githubin kanssa on varsin saumaton). Linkitin sitten myös VS coden WSL:ään joten saan kirjoitettua palautukset VS codella ja pushattua ne WSL:stä suoraan Githubiin. Mermaid ei ollut tämän tehtävänannon listalla suoraan mutta siitä oli maininta jossain toisessa dokumentaatiossa (joita oli liian paljon ja liian monessa paikassa että niiden seuraaminen ja päättäminen että mitä uskoo oli ongelma).

Tehtävässä käytin Chatgpt:tä apuna pitämään yllä osoite-listaa. Annoin kehoitteeksi "Älä vastaa nyt seuraaviin, kirjoitan vain itselle muistiin tähän ip-osoitteita/laitteita mitä löydän" jonka jälkeen pastesin käytettävien komentojen esim. nmap --traceroute -sn 10.10.10.0/24 tulosteita sille. Lopuksi käytin kehoitetta "Tee selvästi luettava taulukko ip-osoitteista pastetuista kehoitteista". Lopuksi, kunhan sain tehtyä omasta mielestäni selkeän reititys-taulukon/topologian niin pastesin sen chatgpt:lle ja annoin kehoitteen "Tee annetusta topologiasta Mermaidille tehty koodi käytettäväksi Githubissa". 

Containerlabin topologiasta tehdyssä kuvassa en enään käyttänyt Chatgpt:n apua kun huomasin että mermaidissa käytetty koodi oli hyvinkin yksinkertaista ja sain tehtyä paremman itse.

Tehtävänannossa oli tehtävään arvioitu käytettävä aika 4-8h joka vähintään tuplaantui minun tapauksessa. 

### Miten dokumentaatio auttaa IT-asiantuntijaa?

Ajantasainen dokumentaatio helpottaa verkon ylläpitoa ja vianetsintää. Tämä olisi ehdottoman tärkeää aloittaa heti verkkoa suunnitellessa tekemään jolloin kaikki, esimerkiksi laitteiden vaihdot, lisäykset tai poistot olisi huomattavasti helpompia ja kokonaisuuden hallinta pysyy helpommin käsissä jos verkko lähtee laajentumaan.