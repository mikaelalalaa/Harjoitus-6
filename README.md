# Harjoitus-6


Tehtävät löytyvät opettajamme [Tero Karvisen sivuilta](https://terokarvinen.com/2021/configuration-management-systems-palvelinten-hallinta-ict4tn022-2021-autumn/#h6-ikkuna-ja-projekti)

## z) Lue ja tiivistä 

[Control Windows with Salt](https://terokarvinen.com/2018/control-windows-with-salt/)


[Configure Windows and Linux with Single Salt Module](https://terokarvinen.com/2018/configure-windows-and-linux-with-salt-jinja-if-else-and-grains/)


[Windows Package Manager](https://docs.saltproject.io/en/latest/topics/windows/windows-package-manager.html)



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



## d) Tee omalle miniprojektille weppisivu


