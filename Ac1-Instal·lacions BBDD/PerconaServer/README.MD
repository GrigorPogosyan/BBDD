# Instal·lació Percona Server <img align="right" width="200" src="captures/percona_logo.png"/>

### Connexió per SSH al Sistema
Primer de tot, ens connectarem a la màquina per **SSH** desde CMD (ho tenim prèviament instal·lat).
```
ssh machineuser@ip 
```
<img src="captures/ssh.png">


### Actualització dels Paquets del Sistema
Després, actualitzarem els nostres paquets del sistema.
```
yum update && yum upgrade
```
<img src="captures/upgrade.png">

### Instal·lació des del Repositori
1. Començarem instal·lant el Repositori de Percona Server per MySQL amb la següent comanda amb un usuari **root** o **sudo**.
```
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```
<img src="captures/install_percona.png"/>
&nbsp;

2. Activarem el Repositori Instal·lat de Percona.
```
sudo percona-release setup ps80 -y
```
<img src="captures/enable_repository.png"/>
&nbsp;

3. 




