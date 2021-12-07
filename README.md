# Harjoitus-6


Tehtävät löytyvät opettajamme [Tero Karvisen sivuilta](https://terokarvinen.com/2021/configuration-management-systems-palvelinten-hallinta-ict4tn022-2021-autumn/#h6-ikkuna-ja-projekti)

## z) Lue ja tiivistä 

[Control Windows with Salt](https://terokarvinen.com/2018/control-windows-with-salt/)

* Salt masterilla pitää olla sama tai uudempi versio, kuin minionilla.
*  windowsin Software Repositories voi sallia komennoilla:
 ```
sudo mkdir /srv/salt/win
sudo chown root.salt /srv/salt/win
sudo chmod ug+rwx /srv/salt/win
 ```
 * Käynnissä olevien työtten statuksen voi katsoa komennolla `sudo salt-run jobs.active`


[Configure Windows and Linux with Single Salt Module](https://terokarvinen.com/2018/configure-windows-and-linux-with-salt-jinja-if-else-and-grains/)

* jos windosin ja linuxin hakemistot ovat erillaisia, niin voi luoda salt moduulin joka toimii monessa käyttöjärjestelmässä. 
  * laitetaan ehdollinen lokiikka sls tiedoston alkuun
  * katsotaan YAML 
  * jos linuxilla ja windowsilla ei ole mitään yhtistä, niin kirjoitetaan erilliset moduulit

[Windows Package Manager](https://docs.saltproject.io/en/latest/topics/windows/windows-package-manager.html)

windowsin paketti manageri joka tarjoaa paketti managerin samankaltaisia kun `yum` ja `apt` linuxsissa. Saltstack on luonut windowsille oman GitHub repositoryn löytyy osoitteesta: (salt-winrepo-ng)[https://github.com/saltstack/salt-winrepo-ng]

winrepo voi nopeasti ottaan käyttöön komennoilla

*master*

```
salt-run winrepo.update_git_repos
salt * pkg.refresh_db
salt * pkg.install firefox_x64
```

*minion*

```
salt-call --local winrepo.update_git_repos
salt-call --local pkg.refresh_db
salt-call --local pkg.install firefox_x64
```
Konfiguraatio tiedosto löytyy

* Linux masterilla: /srv/salt/win/repo-ng (salt://win/repo-ng)
* ilman masteria: C:\salt\srv\salt\win\repo-ng (salt://win/repo-ng)

## a) Kerää tietoa Windowsista Saltilla.

Olin jo tunnilla asentanut saltin windowsiin pääsin heti keräämään jotain tieoja. Koska pelkällä `sudo salt 'winn' grains.items` komennolla tulee paljon tietoa alla olevassa kuvassa näkyy puolet tuloksesta.  

![image](https://user-images.githubusercontent.com/93308960/144883071-67be5556-e68d-4383-8398-dba029144cfa.png)

Joten kävin tuloksen läpi ja valitsin neljä kinnostavinta `osfullname cpu_model cwd virtual`

Komentona toimi 
```
sudo salt 'winn' grains.item osfullname cpu_model cwd virtual

```

Tulokset näkyvät alla olevassa kuvassa 

![image](https://user-images.githubusercontent.com/93308960/144847916-b0705c3f-541a-4741-83a8-376de7687f13.png)

Hieman selvennystä mitä tuloksena tuli

* cpu_model kertoo prosessorin 
* cwd kertoo minionin hakemisto polun
* osfullname kertoo käyttöjärjestelmän koko nimen, tässä tapauksessa minionin käyttöjärjestelmä oli microsoft windows 10 home
* virtual kertoo onko kone fyysinen tai virtuaali, tässä minion oli fyysinen kone


## b) Asenna ohjelmia Windowsille Saltilla.

Loin hakemiston `srv/salt/winn` johon tein `init.sls` tiedoston. 

Tiedostoon lisäsin alla olevan tekstin
 
```
git:
  pkg.installed

micro:
  pkg.installed
```
Tämän jälkeen ajoin komennon `sudo salt 'winn' state.apply winn`

Kuvassa näkyy että git saatiin asennettua mutta micro asennuksen kanssa tuli ongelmia.
Micron pakettia ei läydetty joten sen takia se ei asentunut

![image](https://user-images.githubusercontent.com/93308960/144851126-1043e979-4199-4119-b164-fac40da6b75e.png)

Päätin sitten muokata init tiedostoa ja vaihtaa micro sovelluksen tilalle vlc sovellus.

```
git:
  pkg.installed

vlc:
  pkg.installed
```
Sitten ajoin `state.apply` komennon uudestaan.

Kuten alla olevasta kuvasta näkyy vlc asennus onnisui, koska git oli asennettu jo aikasemmin nii muita muutoksia ei tullut.

![image](https://user-images.githubusercontent.com/93308960/144857886-c98349e8-fe63-47be-9e0d-e2bb51379906.png)


Kuvasta vielä nätkyy windowsin Program tiedostosa että sovellukset asentui.

![image](https://user-images.githubusercontent.com/93308960/144862133-4e6d2ebb-a833-43c7-9bea-c8c6f07861c7.png)


## c) Draft zero.

Tämä ei ole vielä virallinen projekti päätös mutta backup. Eli olen tekemässä apache2 pohjalta nettisivua, käytän infraa koodina hyödyksi sovellusten lataamiseen ja konfigurointiin. Otan muutoksen käyttöön saltin avulla.

Aloitin tekemällä uuden debian-bullseye virtuaalikoneen ja siihen asensin salt master & minion. 

Komento oli:
```
sudo apt-get install salt-minion
&
sudo apt-get install salt-master
```
Tämän jälkeen katsoin masterin ip-osoitteen ja laitoin tiedon /etc/salt/minion hakemistoon. Sitten käynnistin minionin uudelleen komennolla `sudo systemctl restart salt-minion`, katsoin oliko avain pyytö tullut masterille komennolla `sudo salt-key`. Oli tullut ja hyväksyin sen komennolla `sudo salt-key -A`

![image](https://user-images.githubusercontent.com/93308960/145073179-507690d1-0e4e-472a-89a0-720d8f8b4c75.png)

Testasin minion-master yhteyttä komennolla `sudo salt '*' test.ping`  ja yhteys oli onnistunut

![image](https://user-images.githubusercontent.com/93308960/145073234-08c90899-a67e-48e0-9684-c31bed3e3e39.png)

Yhteyden luomisen jälkeen loin uuden hakemiston `srv/salt/allaps` ja tein tiedoston init.sls. Sinne kirjoitin sovellukset jotka tarvitsen eli:

* apache2
* openssh-server
* net-tools
* tree

![image](https://user-images.githubusercontent.com/93308960/145077502-2dee1508-c5d1-4b35-91e7-cec7417ab598.png)

Tallensin tiedoston ja ajoin komennon `sudo salt '*' state.apply allaps`, kuten alla olevasta kuvasta näkyy kaikki asennettiin onnistuneestin

![image](https://user-images.githubusercontent.com/93308960/145077291-625c37a4-0c54-4f9d-9104-83b7c1e827ae.png)

Sitten loin uuden hakemiston `srv/salt/apache2` johon loin init.sls tiedoston ja alla olevasta kuvasta näkyy mitä laitoin.

Eli katsoin että apache on asennettu ja että se on käynnissä *(ensimmäinen ja vika rivi)*, sitten korvasin alkuperäsien index.html tiedoston omalla index.html tiedostolla *(toiseksi ylin)* ja yritin vielä saada palomuuriin reiän apache2 sovellukselle *(tokavika rivi)*.

![image](https://user-images.githubusercontent.com/93308960/145083899-86aeb9bc-c4f8-438c-b0ee-286a6ecca71c.png)

Ajoin komennon `sudo salt '*' state.apply apache2`

![image](https://user-images.githubusercontent.com/93308960/145079774-a0af2ddf-e9e4-4efc-90ac-a756b190d81d.png)


![image](https://user-images.githubusercontent.com/93308960/145079811-d3bc5068-3d52-4ee8-9c9c-f323fba6943c.png)

Kuten yllä olevista kuvistä näkyy kaikki muut muutokset onnistui paitsi palomuuri asetus, en vielä tällä hetkellä osannut korjaa asiaa.
Mutta kuten alla olevasta kuvasta näkyy pystyn näkemään selaimesta sivuni

![image](https://user-images.githubusercontent.com/93308960/145080851-76fc8e23-f9f6-40aa-b990-f0c7dc64cb9d.png)



## d) Tee omalle miniprojektille weppisivu

Linnki miniprojektin pohjaan https://github.com/mikaelalalaa/Harjoitus-7


## lähteet 

https://terokarvinen.com/2021/configuration-management-systems-palvelinten-hallinta-ict4tn022-2021-autumn/#h6-ikkuna-ja-projekti

https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/

https://terokarvinen.com/2017/04/11/vagrant-revisited-install-boot-new-virtual-machine-in-31-seconds/

https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/

https://docs.saltproject.io/en/latest/topics/windows/windows-package-manager.html
