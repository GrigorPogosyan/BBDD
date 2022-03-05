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

3. Instal·lem els paquets (Percona Server)
```
sudo yum install percona-server-server
```
<img src="captures/install_packages.png"/>
&nbsp;...
<img src="captures/install_packages2.png">
&nbsp;

### Post Instal·lació
Al acabar la instal·lació cal que ferm un seguit de passos per acabar de tenir correctament modificat i montar el nostre Percona Server

1. Primer de tot encendrem el servei de MySQL.
```
sudo systemctl start mysqld
```
<details open>
<summary>Screenshot</summary>
<img src="captures/start_mysqld.png">
</details>

2. Posteriorment configurarem que MySQL s'inicii al iniciar el Sistema operatiu
```
sudo systemctl enable mysqld
```
3. Farem el secure_installation per **millorar la seguretat** del nostre servidor, el Script que executarem fa lo següent:
    - Canvia la contrassenya **Root**.
    - **Inhabilita la connexió** remota a la BBDD per l'usuari **Root**.
    - Elimina els usuaris anónims.
    - Elimina la Base de Dades de Test.
    - Reinicia els permisos de les Taules.
```
mysql_secure_installation
```




