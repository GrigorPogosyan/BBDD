# Instal·lació MySQL 8.0 <img align="right" width="130" src="../imatges/mysql_logo.png"/>

## Connexió per SSH al Sistema
Primer de tot, ens connectarem a la màquina per **SSH** desde CMD (ho tenim prèviament instal·lat).
```
ssh machineuser@ip 
```
<details open>
<summary>Screenshot <b>(Connection SSH)</b></summary>
<img src="captures/ssh.png">
</details>

## Actualització dels Paquets del Sistema
Després, actualitzarem els nostres paquets del nostre sistema.
```
# yum update && yum upgrade
```
<details open>
<summary>Screenshot <b>(OS Upgrade)</b></summary>
<img src="captures/upgrade.png">
</details>
<hr>

## Instal·lació des del Repositori
1. Començarem instal·lant el Repositori de MySQL 8.0 amb la següent comanda amb un usuari **root** o amb**sudo**.
```
# sudo yum install https://repo.mysql.com/mysql80-community-release-el8-3.noarch.rpm
```
<details open>
<summary>Screenshot <b>(Instal)</b></summary>
<img src="captures/install_percona.png">
</details>
