# Viikko 4 – Ansible ja Infrastructure as Code (v1.01)

## 1. Johdanto

Infrastructure as Code (IaC) tarkoittaa infrastruktuurin/koneryhmien määrittelyä, tekoa ja hallintaa koodin tai kirjoitettujen kehoitteiden avulla ilman käsin tehtävää asentelua/määrittelyä. IaC poistaa käsin tehtäviä virheitä ja nopeuttaa isojen kokonaisuuksien hallintaa kun voidaan ajaa samat määrittelyt niin pienille kuin isoillekin ryhmille kerralla kunhan ajettavat määritystiedostot ovat huolellisesti testatut ettei monisteta virhettä kaikkiin.

Ansible on avoimen lähdekoodin työkalu jonka avulla voidaan toteuttaa IaC periaatetta. Red Hat omistaa Ansible - tuotemerkin, tukee laajasti ansiblen kehitystä ja tarjoaa omaa Ansible automaatioalustaa maksullisella tilauksella mutta avoimeen lähdekoodiin perustuva ansible on kaikkien vapaasti käytettävä. Ansible rakentuu vahvasti inventoryn ja playbookkien ympärille mutta ansiblea voidaan käyttää myös ilman niitä.
Inventoryllä voidaan määritellä ja jaotella hallittavat koneet ja laitteet jolloin voidaan ajaa tarvittavia komentoja/suoritteita (playbookkeja) kohdennetusti joko kaikille tai esimerkiksi pelkästään jollekin tietylle osiolle/kohteelle joka on määritelty inventoryssä.

## 2. Inventory

Tällä viikolla tutustuttiin kontti-ympäristössä ansiblen inventoryyn. Se oli rakennettu osioittain jossa oli reitittimet, työasemat, serverit ja valvontakohteet osioitu ja määritelty niiden muuttujat. Lisäksi oli osioitu verkon ja segmenttien mukaan sekä jaoteltu järjestelmän mukaisesti. 
Tämä mahdollistaa kohdennetun määrittelyn, eli jos halutaan vain pelkät reitittimet tai pelkät ubuntu-kohteet määritellä niin se onnistuu. Tai vielä pidemmälle jos laajennetaan inventoryä käsittämään vaikka windows-kohteita jotka on jaoteltu myös työasemien ja servereiden alle muiden kanssa niin voidaan kohdentaa pelkästään työasemiin jotka sisältävät kaikki windows-järjestelmän.

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

saatiin haluttu lopputulos. Sen lisäksi piti muokata handlers - kohtaa vastaamaan tätä.

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

Playbookin toteutin samalla tyylillä kuin snmp:n playbookin mutta käytin vain pelkästään hostina web1:stä. Sivun muokkauksessa kopioin vain inventory_hostnamen korostettuna h1:ksi. Tämä siksi että siitä selviää heti asennuskohteen nimi jos esimerkiksi playbookkia ajetaan asentamaan useita web-palvelimia. 

### 4.1 Playbookin rakenne

```bash
---
- name: Install apache web server
  hosts: web1
  become: true

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
    
    - name: Install Apache2
      apt:
        name: apache2
        state: present

    - name: Start and enable Apache2 service
      ansible.builtin.sysvinit:
        name: apache2
        state: started
        enabled: yes

    - name: Print hostname to index.html
      copy:
        dest: /var/www/html/index.html
        content: "<h1> {{ inventory_hostname }} </h1>"
```

### 4.2 Playbookin suorittaminen ja testaus

Playbookin onnistunut asennus

[Install-webserver playbookin ajo](images/apache2-playbook-install.png)

ja testaus client1:seltä

[Webserver testaus client1:stä](images/apache2-test_from_client1.png)

## 5. Järjestelmätietojen kerääminen

Kokeilin näitä kerätä ensin ansiblen setupilla ja sen perään kokeilin debugilla mutta kun edellisessä kohdassa kokeiltiin kirjoittaa playbookki tätä varten niin kirjoittelin display-data - playbookin jolla saan kaikki kerralla.

**display-data.yml**

```bash
---
- name: Display data for week04
  hosts: all
  gather_facts: true
  tasks:
    - name: Display system information
      ansible.builtin.debug:
        msg:
          - "Host: {{ inventory_hostname }}"
          - "IP: {{ ansible_default_ipv4.address }}"
          - "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
          - "CPU cores: {{ ansible_processor_cores }}"
          - "Memory: {{ ansible_memtotal_mb }} MB"
```

Näin sain helposti kerättyä vaaditut tiedot taulukkoa varten

| Name | IP | OS | CPU Cores | Memory |
|---|---|---|---|---|
| client1 | 10.10.10.101 | Ubuntu 24.04 | 16 | 30901 MB |
| attacker | 10.10.10.200 | Kali 2026.3 | 16 | 30901 MB |
| web1 | 10.10.20.101 | Ubuntu 24.04 | 16 | 30901 MB |
| db1 | 10.10.20.102 | Ubuntu 24.04 | 16 | 30901 MB |
| branch-client | 10.10.30.101 | Ubuntu 24.04 | 16 | 30901 |

## 6. Analyysi

Tämän viikon jälkeen ei enää tulisi mieleen alkaa keräämään käsin mitään tietoja tai asentamaan useille koneille. Keskitetysti tehdyillä asennuksilla säästetään huomattava määrä aikaa, saadaan varmuuskopioitua verkkolaitteiden konfiguraatioita ja palautettua ne tai asennettua yksittäisiin kohteisiin joku haluttu versio. 

Usein myös halutaan pysyä joissain tiettyjen ohjelmien versioissa joten tällä myös estetään virheet pakettien asennuksessa kun jokaiseen saadaan se haluttu versio paketista joka ei välttämättä ole se uusin.