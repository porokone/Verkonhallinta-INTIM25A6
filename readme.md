# Verkonhallinta (3 op)

Tervetuloa Verkonhallinta-opintojaksolle.
Nykyaikainen verkonhallinta on siirtynyt voimakkaasti kohti:

automaatiota ja Infrastructure as Code -ajattelua
jatkuvaa telemetriaa (streaming telemetry)
API-pohjaista hallintaa
keskitettyä havainnointia (observability)
pilvi- ja hybridiverkkojen hallintaa
tietoturvan ja verkonhallinnan yhdistymistä

Kurssilla käytetään yhtenäistä virtuaalista verkkoympäristöä, joka simuloi yritysverkkoa. Kaikki harjoitukset suoritetaan saman ympäristön päälle lisäämällä siihen uusia palveluita, valvontaa ja automaatiota kurssin edetessä.

Kurssin aikana käytetään seuraavia teknologioita:

- [Containerlab](https://containerlab.dev/)
- [Docker](https://www.docker.com/)
- Linux
- [FRRouting](https://frrouting.org/)
- [SNMP](https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol)
- [Syslog](https://en.wikipedia.org/wiki/Syslog)
- [Prometheus](https://prometheus.io/)
- [Grafana](https://grafana.com/oss/grafana/)
- [Zabbix](https://www.zabbix.com/)
- [Ansible](https://docs.ansible.com/projects/ansible/latest/getting_started/introduction.html)
- [NetBox](https://netboxlabs.com/docs/netbox/)
- [Wireshark](https://www.wireshark.org/)

---

# Oppimistavoitteet

Opintojakson jälkeen osaat:

- Dokumentoida verkkoympäristön
- Kerätä ja analysoida SNMP-dataa
- Määrittää keskitetyn lokienhallinnan
- Rakentaa monitorointijärjestelmän
- Toteuttaa verkkolaitteiden automaatiota
- Dokumentoida ympäristön NetBoxilla
- Tunnistaa ja analysoida verkkovikoja
- Toteuttaa perusvalvonnan Zabbixilla

---

# Kurssin ympäristö

Topologia muodostaa kuvitteellisen yritysverkon:

```text
                    r1-edge
                       |
                    r2-core
        ---------------+----------------
        |              |               |
     Users          Servers         Management

        |              |               |
     client1         web1         ansible
     client2         db1          grafana
                                   zabbix
                                   prometheus
                                   syslog
                                   netbox

                       |
                   r3-branch
                       |
                branch-client
```

---

# Esivaatimukset

Tarvitset tietokoneen, jossa vähintään 16Gt muistia sekä oikeuden asentaa koneeseen uusia työkaluja. Ympäristö hyötyy suuremmasta muistinmäärästä. 

## Windows

Suositeltu:

- Windows 11
- WSL2
- Ubuntu 24.04

Pakolliset ohjelmistot:

- Docker Desktop tai Docker Engine ce asennettuna wsl ympäristöön
- Git
- Visual Studio Code

## Linux

Suositeltu:

- Ubuntu 24.04

Pakolliset ohjelmistot:

- Docker Engine
- Git
- Visual Studio Code

---

# Asennus

## 1. Asenna Git

Ubuntu:

```bash
sudo apt update
sudo apt install git -y
```

Tarkista:

```bash
git --version
```

---
## 2. Asenna Containerlab

huomaa, tämä asentaa docker engine CEn koneelle, jollei koneelta löydy docker binääriä

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

Tarkista asennus:

```bash
containerlab version
```

---

## 3. Varmista Docker asennuksen olemassa olo

Tarkista toiminta:

```bash
docker version
```

---

## 4. Kloonaa kurssirepositorio

```bash
git clone <kurssin_git_repository>
```

```bash
cd Verkonhallinta
```

---

# Ympäristön käynnistys

Käynnistä ympäristö:

```bash
sudo scripts/deploy.sh
```

Tarkista tila:

```bash
sudo scripts/status.sh
```

---

# Hallintapalvelujen osoitteet

Kun ympäristö on käynnissä, palvelut löytyvät selaimella seuraavista osoitteista.

## Grafana

```text
http://localhost:3000
```

## Prometheus

```text
http://localhost:9090
```

## Zabbix

```text
http://localhost:8080
```

## NetBox

```text
http://localhost:8000
```

NetBoxiin on lisatty `Topology Views` -plugin visuaalista topologian piirtoa varten.

NetBox ajetaan tassa ymparistossa erillisena Docker Compose -pinona (ei containerlab-solmuna).
Pino liitetaan automaattisesti `clab-mgmt`-hallintaverkkoon, jotta NetBox toimii Source of Truth -palveluna koko topologialle.

---

# Ensimmäinen yhteystesti

Avaa shell client1-konttiin:

```bash
docker exec -it clab-hamk-verkonhallinta-golden-client1 bash
```

Testaa yhteys palvelimeen:

```bash
ping 10.10.20.101
```

Tarkista reitti:

```bash
traceroute 10.10.30.101
```

---

# Kurssin tehtävien eteneminen

## Viikko 1

Verkon dokumentointi

Tee:

- Topologiakuva
- IP-suunnitelma
- NetBox-dokumentaatio

---

## Viikko 2

Vikojenhallinta

Tee:

- Yhteystestaus
- Reittien tutkiminen
- Vian syyn selvittäminen

---

## Viikko 3

SNMP ja Syslog

Tee:

- SNMP-agentin käyttöönotto
- Lokien keskitys
- Hälytysten tutkiminen

---

## Viikko 4

Prometheus ja Grafana

Tee:

- Monitorointi
- Dashboardit
- Kapasiteettianalyysi

---

## Viikko 5

Ansible

Tee:

- Automaattinen konfigurointi
- Varmuuskopiot
- Muutosten hallinta

---

## Viikko 6

Tietoturva

Tee:

- Verkkoliikenteen analysointi
- Wireshark
- Lokianalyysi

---

## Viikko 7

Zabbix

Tee:

- Hostien lisääminen
- Triggerit
- Dashboardit
- Raportointi

---

# Topologian käyttö

Ensimmaisella kerralla luo NetBoxin ymparistotiedosto:

```bash
cp configs/netbox/.env.example configs/netbox/.env
```

Paivita salasanat tiedostoon `configs/netbox/.env`.

```bash
bash scripts/deploy.sh
bash scripts/status.sh
bash scripts/netbox-seed.sh
bash scripts/reset.sh
bash scripts/destroy.sh
```

Muistiraportti (Windows + WSL + Docker):

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/memory-report.ps1
```

Labran konttimetriikoiden tila (Prometheus + Grafana):

```bash
bash scripts/lab-monitoring-status.sh
```

Huom: konttimetriikat ovat olemassa vain labran ollessa kaynnissa, koska cAdvisor on osa topologiaa.
Kun labra sammutetaan `destroy.sh`:lla, metrikat ja targetit poistuvat samalla.

# SSH ja komentorivi

Voit avata minkä tahansa kontin shellin:

```bash
docker exec -it <container> bash
```

Esimerkiksi:

```bash
docker exec -it clab-hamk-verkonhallinta-golden-r2-core bash
```

---

# Topologian visualisointi

Voit generoida topologiakuvan:

```bash
containerlab graph -t golden.clab.yml
```

---

# Ympäristön sulkeminen

Kun lopetat työskentelyn:

```bash
containerlab destroy -t golden.clab.yml --cleanup
```

---

# Vianetsintä

## Docker ei käynnisty

Tarkista:

```bash
docker ps
```

---

## Containerlab ei löydä topologiaa

Tarkista nykyinen hakemisto:

```bash
pwd
ls
```

---

## Kontti ei käynnisty

Tutki lokit:

```bash
docker logs <container_name>
```

---

# Hyvät käytännöt

Kurssilla suositellaan käyttämään Git-versionhallintaa.

Tallenna:

- Ansible-playbookit
- Konfiguraatiot
- Dokumentaatiot
- Raportit

omaan Git-repositorioon koko kurssin ajan.

---

# Lisenssit

Kurssi käyttää avoimen lähdekoodin ohjelmistoja.

- Docker
- Containerlab
- FRRouting
- Grafana
- Prometheus
- Zabbix
- NetBox
- Ansible

Lisenssiehdot löytyvät kunkin ohjelmiston omilta verkkosivuilta.