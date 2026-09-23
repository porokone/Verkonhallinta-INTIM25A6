# Viikko 4 – Ansible ja Infrastructure as Code (v0.4)

## 1. Johdanto

Infrastructure as Code (IaC) tarkoittaa infrastruktuurin/koneryhmien määrittelyä, tekoa ja hallintaa koodin tai kirjoitettujen kehoitteiden avulla ilman käsin tehtävää asentelua/määrittelyä. IaC poistaa käsin tehtäviä virheitä ja nopeuttaa isojen kokonaisuuksien hallintaa kun voidaan ajaa samat määrittelyt niin pienille kuin isoillekin ryhmille kerralla.

Ansible on avoimen lähdekoodin työkalu jonka avulla voidaan toteuttaa IaC periaatetta. Red Hat omistaa Ansible - tuotemerkin, tukee laajasti ansiblen kehitystä ja tarjoaa omaa Ansible automaatioalustaa maksullisella tilauksella mutta avoimeen lähdekoodiin perustuva ansible on kaikkien vapaasti käytettävä. Ansible rakentuu vahvasti inventoryn ja playbookkien ympärille mutta ansiblea voidaan käyttää myös ilman niitä.
Inventoryllä voidaan määritellä ja jaotella hallittavat koneet ja laitteet jolloin voidaan ajaa tarvittavia komentoja/suoritteita (playbookkeja) kohdennetusti joko kaikille tai esimerkiksi pelkästään jollekin tietylle osiolle/kohteelle joka on määritelty inventoryssä.

## 2. Inventory

Tällä viikolla tutustuttiin kontti-ympäristössä ansiblen inventoryyn. Se oli rakennettu osioittain jossa oli reitittimet, työasemat, serverit ja valvontakohteet osioitu ja määritelty niiden muuttujat. Lisäksi oli osioitu verkon ja segmenttien mukaan sekä jaoteltu järjestelmän mukaisesti. 
Tämä mahdollistaa kohdennetun määrittelyn, eli jos halutaan vain pelkät reitittimet tai pelkät ubuntu-kohteet määritellä niin se onnistuu. Tai vielä pidemmälle jos laajennetaan intentoryä käsittämään vaikka windows-kohteet jotka on jaoteltu työasemien ja servereiden alle niin voidaan kohdentaa pelkästään työasemiin jotka sisältävät kaikki windows-järjestelmän.

Puhtaasti voidaan säästää aikaa ja pienentää virhemarginaalia mutta se vaatii aina testaamista että se virhe ei koske jokaista laitetta.

## 3. Esimerkkiplaybookit

Esimerkkinä oli kolme playbookkia jotka testattiin. 

### 3.1 Ping

Ensimmäisenä ajettin ping-playbookit jolla testattiin yhteyksiä. Siinä käytettiin inventoryssä olevia määrittelyjä hyväksi koska esimerkiksi palvelimet ja reitittimet vaativat eri määrittelyt vastatakseen.

[Kuvakaappaus ping-playbookista](images/ping.png)

### 3.2 Install-SNMP

Toisena suoritettiin install-SNMP - playbookki mutta se login mukaan asentui mutta virhe-ilmoitukset tulivat kun sen piti laittaa SNMP-palvelu päälle.

[Vikailmoitus install-SNMP-playbookista](images/snmp-install-fail.png)

Se myös antoi suoran viittauksen missä sen vika voisi olla eli aloin etsimään vikaa kyseisestä osiosta

```bash
    - name: Enable SNMP service
      service:
        name: snmpd
        enabled: yes
        state: started
```

Tämän pitäisi olla oikein mutta hieman tutkittuani asiaa sain selville että kontti-ympäristössä tuo ansiblen service ei välttämättä aina toimi. Tämän myös varmistin kohteita tutkittuani kun selvitin mikä prosessi on PID 1, eli tässä tapauksessa saimme vastaukseksi 'bash' eli kontit eivät käyttäneet systemd:tä järjestelmänään. Tätä selvitellessäni en halunnut päätyä suoraan shell-komentoja vaan halusin nimenomaan käyttää palveluita ja päädyin kokeilemaan sysvinit:iä.
Eli muokkaamalla kohtaa muotoon 

```bash
    - name: Enable SNMP service
      ansible.builtin.sysvinit:
        name: snmpd
        enabled: yes
        state: started
```

Saatiin haluttu lopputulos. Sen lisäksi piti muokata handlers - kohtaa vastaamaan tätä.

[Install-SNMP-playbookin onnistunut ajo](images/snmp-install-hit.png)

### 3.3 Install-node-exporter

Tämän ajo onnitui ensimmäisellä kerralla. Mitään ongelmia ei ollut.

[Install-node-exporter - playbookin onnistunut asennus](images/node-exporter-hit.png)

Mutta miksi tämä onnistui mutta SNMP ei? Syy löytyy tavasta jolla se ajetaan ylös. Kun SNMP yritettiin käynnistää niin käytettiin ansiblen service-palvelua mutta node-exportterin osalta tapetaan kyseinen prosessi ja nostetaan se ylös nohupilla

```bash
    - name: Start node_exporter
      shell: |
        pkill node_exporter || true
        nohup /opt/node_exporter/node_exporter > /tmp/node_exporter.log 2>&1 &
      args:
        executable: /bin/bash
```

eli käytetään kehoitteita joita pyrin välttämään snmp:n kohdalla.

## 4. Oma playbook

Oman palvelimen asennuksen ja konfiguroinnin automatisointi Ansiblen avulla.

### 4.1 Playbookin rakenne

### 4.2 Playbookin suorittaminen ja testaus


## 5. Järjestelmätietojen kerääminen

Järjestelmätietojen kerääminen Ansible setup -moduulilla.


## 6. Manuaalisen ja automatisoidun asennuksen vertailu

Vertailu käsin tehdyn asennuksen ja Ansible-automaation välillä.


## 7. Yhteenveto ja pohdinta

Yhteenveto harjoituksesta, opituista asioista, hyödyistä ja mahdollisista ongelmista.