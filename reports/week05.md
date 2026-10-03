# Viikko 5 – Wireshark ja verkkoliikenteen analysointi (v1.01)

## Osa 1 – Yle ja GeoIP

Tehtävänä oli selvittää minne ylen käyttämän palvelun webbi-palvelin sijoittuu. Vaihtoehtoina oli käyttää capture filtteriä tai display filtteriä mutta päädyin kokeilemaan molempia. Ennen tehtävän alkua kävin läpi Chris Greerin youtube-kanavallaan julkaisseen sarjan ja lesson 10:ssä latasin jo koneelle GeoIP Liten tietokannat jonka avulla selvitin kohteen sijainnin.

Ylen verkkosivun liikennettä kaapattiin Wiresharkilla ja liikennettä tutkittiin Display Filterillä:

`dns.qry.name contains "yle"`

Ylen liikenne ohjautui Amazon CloudFront -palvelimille. Yksi kaappauksessa havaittu palvelin oli `18.165.122.16`, jonka nimi oli `server-18-165-122-16.hel51.r.cloudfront.net`. Wiresharkin GeoIP-tietokanta ilmoitti sijainniksi Yhdysvallat ja organisaatioksi Amazon.com Inc.

Liikennettä rajattiin Display Filterillä:

`ip.addr == 18.165.122.16`

Capture Filteriä kokeiltiin erillisessä kaappauksessa kun olin saanut jo selville display filtterin avulla kohteen ip-osoitteen:

`host 18.165.122.16`

Capture Filter rajasi jo kaapattavan liikenteen kyseiseen palvelimeen, kun taas Display Filterillä rajattiin jälkikäteen jo kaapattua liikennettä.

[Capture filtterillä kaapattuja paketteja Wiresharkissa](images/yle-capture-filtter.png)

## Osa 2 – DNS ja TLS

Isoimmat ongelmat kohtasin tämän osion kanssa. Ennen tehtävän onnistunutta suoritusta jouduin tyhjentämään käytettävän koneen dns-muistin komennolla

```bash
ipconfig /flushdns
```

ja sen jälkeen jouduin oman reitittimen dns-muistin tyhjentämään koska en saanut dns-paketteja kaapattua learn.hamk.fi - osoitteesta ollenkaan.

`learn.hamk.fi` DNS A-kyselyn vastauksena saatiin:

- IP-osoite: `195.148.239.84`
- TTL: `844 s` (14 min 4 s)

`hamk.fi`-alueen authoritatiiviset DNS-palvelimet selvitettiin NS-kyselyllä:

```bash
nslookup -type=NS hamk.fi
```

- `ns1.hamk.fi`
- `ns2.hamk.fi`
- `ns3.hamk.fi`
- `ns-secondary.funet.fi`

TLS-kättelyn Client Hello -viestissä selain tarjosi TLS 1.3- ja TLS 1.2 -versioita sekä 15 varsinaista cipher suitea (+ GREASE). Tarjolla olivat mm. `TLS_AES_128_GCM_SHA256`, `TLS_AES_256_GCM_SHA384` ja `TLS_CHACHA20_POLY1305_SHA256`.

Server Hello -viestissä palvelin valitsi:

- TLS-versio: **TLS 1.3**
- Cipher suite: `TLS_AES_128_GCM_SHA256 (0x1301)`

## Osa 3 – HTTP Basic Authentication

Docker-kontin HTTP-liikenne kaapattiin tcpdumpilla ja avattiin Wiresharkissa. Liikenne rajattiin Display Filterillä:

`http`

HTTP Basic Authentication välittää tunnukset muodossa `käyttäjätunnus:salasana` Base64-koodattuna. Base64 ei ole salaus, joten salaamattomasta HTTP-liikenteestä tunnukset voitiin selvittää. Basic Auth - metodi käsittää yleensä pyynnön clientilta serverille, serveriltä tulee Unauthorized tieto clientille, clientti sitten lähettää credentialsit serverille ja serveri lähettää sivun jos ne ovat oikeat. Tuosta oli helppo haarukoida kolmas paketti käsittelyyn.

[Dumpista löytyvä http->get paketti](./images/tcpdump-wireshark-basic-auth.png)

Wiresharkin HTTP-paketin Basic Credentials -tiedoista löytyivät käytetyt tunnukset:

`tl1labra:Qwerty1!`

[Koko tcpdump](./attachments/dump.pcap)