# Viikko 3 – Monitorointi Prometheuksella ja Grafanalla

## 1. Johdanto

Verkonhallinnassa monitoroinnilla tarkoitetaan järjestelmän eri osien ja niiden toimintaa kuvaavien yksittäisten mittareiden jatkuvaa seurantaa. Kyseessä voi olla yksittäisen suorittimen rasituksen seurannasta yksinkertaiseen järjestelmän ylhäällä olo-tietoon.
Eri mittareiden monitorointi on tärkeää erityisesti koko tietoverkon ylläpidon ja kehityksen kannalta. Kun on jatkuvaa tietoa millä tasolla eri yksiköt/laitteet/mittarit toimii niin voidaan keskittyä kehittämään niitä järjestelmän osia jotka ovat lähellä kriittistä toimintapistettä tai ongelmatilanteissa voidaan jo nähdä paikat joissa se ongelma on. Myös mitattaviin kohteisiin voidaan lisätä hälytyksiä jolloin päästään puuttumaan ongelmiin ennen kuin ne näkyvät käyttäjille.

Tällä viikolla tulivat tutuiksi Node exporter, Prometheus, PromQL ja Grafana.

### Node exporter

Node exporter on avoimen lähdekoodin työkalu. Node exporter lukee ja muuttaa datan, esimerkiksi suorittimen/levytilan/verkkoliitäntöjen tietoja metriikka-muotoon ja laittaa sen jakoon josta scraperi voi käydä noutamassa sen datan.

### Prometheus

Prometheus on avoimen lähdekoodin järjestelmä, joka kerää ja tallentaa valvottavista kohteista dataa aikasarjana. Prometheus käyttää ns. scraping-toimintamallia, eli kuvavaasti kaavitaan dataa kohteiden exportereista yleensä HTTP-rajapinnan kautta.

### PromQL

PromQL on Prometheuksen sisäänrakennettu kyselykieli jolla voidaan käsitellä Prometheuksen tallentamaa dataa ja muodostaa siitä tarvittavia suureita halutussa muodossa. Sillä voidaan esimerkiksi hakea jonkun yksittäisen suureen tietoa tai käsitellä sitä jollain tavalla, esimerkiksi summata/jakaa/muuttaa tarvittavaksi.

### Grafana

Grafana on avoimen lähdekoodin datan visualisointiin tarkoitettu ohjelma. Sillä muutetaan ns. raakadata graaffiseen muotoon kuvaajaksi käyttäen muokattavaa kojelautaa johon voidaan koota haluttuja mittareita. Mittareita voidaan käyttää näyttämään esimerkiksi CPU-kuormitusta, muistinkäyttö-astetta tai verkkoliikennettä ja niihin voidaan lisätä hälytykset jos esimerkiksi CPU-kuormitus nousee liian korkeaksi. Myös tilatietoja voidaan liittää siihen ja saada esimerkiksi palvelimen kaatumisesta hälytykset.

## 2. Ympäristön rakentaminen ja Node Exporterin käyttöönotto

### 2.1 Ympäristö

Kokonaisuus rakennetaan jo aiemmin ylös ajettuun kontti-ympäristöön käyttäen aikaisempien viikkojen raportteja kun kontti-ympäristön tallennus ei toimi.

Valvottavaksi kohteeksi valittiin web1-kontti jossa ajettiin erilaisia testejä, joita sitten tulkittiin Grafanan piirtämistä kuvaajista. Asennuksessa huomasin että myös db1 ja client1 tulivat myös osaksi valvontaa.

### 2.2 Node Exporterin asennus

Jotta Node exporter saadaan asennettua, ensin asennetaan wget seuraavalla komennolla

```bash
apt update && apt install wget tar -y
```

Node exportterista haluamme asentaa uusimman version joten käydään seuraavassa osoitteessa tarkistamassa uusin verio

[Node exporter releases](https://github.com/prometheus/node_exporter/releases/)

Jonka jälkeen voidaan ladata seuraavalla komennolla uusin verio-paketti (muuta oikea versio komentoon)

```bash
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-<version>.linux-amd64.tar.gz
```

Seuraavaksi ladattu paketti puretaan komennolla

```bash
tar xvf node_exporter-*.linux-amd64.tar.gz
```

Siirrytään puretun paketin hakemistoon (tarkista versio-numero)

```bash
cd node_exporter-1.12.1.linux-amd64
```

Käynnistetään Node exportter

```bash
./node_exporter
```

Seuraavalla komennolla voidaan tarkistaa, onko kyseisestä exportterista saatavilla tietoja

```bash
curl http://localhost:9100/metrics
```

Jos vastaukseksi saadaan todella pitkä lista jossa on esimerkiksi seuraavanlaista tietoa

![Osakaappaus Node exportter - listasta](images/node_exporter_list.png)

niin Node exportteri on toiminnassa

## 3. Prometheus

### 3.1 Kohteiden lisääminen

- Mitä kohteita Prometheus valvoo?
- Mitä exportereita käytetään?
- Prometheuksen konfiguraatio

### 3.2 Targets

- Targets-sivun tarkastelu
- Kohteiden UP/DOWN-tila
- Havainnot

### 3.3 PromQL-kyselyt

Esitä vähintään kolme PromQL-kyselyä.

Jokaisesta:
- käytetty kysely
- mitä kysely mittaa/palauttaa
- miksi tieto on hyödyllinen
- havainnot tuloksista

## 4. Grafana Dashboard

### 4.1 Dashboardin rakentaminen

- Grafanan yhdistäminen Prometheukseen
- Valitut mittarit
- Miksi juuri nämä mittarit ovat hyödyllisiä?

### 4.2 Dashboard-näkymät

- Vähintään kaksi dashboard-näkymää
- Kuvakaappaukset
- Mitä näkymissä visualisoidaan?
- Koko dashboardin kuvakaappaus

## 5. Kuormitustesti ja havainnot

### 5.1 Kuormitustestin toteutus

- Miten kuormitus tuotettiin?
- Mitä palvelinta/kohdetta kuormitettiin?
- Mitä mittareita seurattiin?

### 5.2 Mittareiden käyttäytyminen

- CPU
- muisti
- verkko
- muut valitut mittarit
- Kuvakaappaukset ennen kuormitusta / kuormituksen aikana

### 5.3 Havainnot monitoroinnista

- Miten kuormitus näkyi Prometheuksessa ja Grafanassa?
- Löytyikö poikkeamia tai muita mielenkiintoisia havaintoja?
- Mitä johtopäätöksiä mittausten perusteella voidaan tehdä?

## 6. SNMP vs. Prometheus

- SNMP:n ja Prometheuksen toimintaperiaatteiden erot
- Push/pull-toimintamallit
- Mitä tietoa kummallakin voidaan kerätä?
- Käyttökohteet
- Vahvuudet ja rajoitukset

Vertailutaulukko ja johtopäätökset.

## 7. Yhteenveto ja pohdinta

- Mitä opit harjoituksesta?
- Mitä monitorointiympäristöstä opittiin käytännössä?
- Mitkä ovat Prometheuksen vahvuudet?
- Millaisiin ympäristöihin Prometheus soveltuu erityisen hyvin?
- Mitä haasteita tai rajoituksia ratkaisussa on?
- Miten Prometheus/Grafana-ratkaisua voisi kehittää edelleen?