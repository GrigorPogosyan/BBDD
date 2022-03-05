# Instal·lació Percona Server <img align="right" width="200" src="captures/percona_logo.png"/>

### Connexió per SSH al Sistema
Primer de tot, ens connectarem a la màquina per **SSH** desde CMD (ho tenim prèviament instal·lat).
```
ssh user@ip 
```
<img src="captures/ssh.png">


### Actualització dels Paquets del Sistema
Primer de tot, actualitzarem els nostres paquets del sistema.
```
yum update && yum upgrade
```
<img src="captures/upgrade.png">

### Instal·lació des del Repositori
Començarem instal·lant el Percona Server per MySQL amb la següent comanda amb un usuari **root** o **sudo**.
```
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```
<img src="captures/install_percona.png">


