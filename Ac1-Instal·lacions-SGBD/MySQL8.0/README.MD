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

## Instal·lació MySQL 8.0 des del Repositori
1. Començarem instal·lant el Repositori de MySQL 8.0 amb la següent comanda amb un usuari **root** o amb **sudo**.
```
# sudo yum install https://repo.mysql.com/mysql80-community-release-el8-3.noarch.rpm -y
```
<details open>
<summary><b>Instal·lant el Repositori de MySQL 8.0</b></summary>
<img src="captures/install_repository_mysql.png">
</details>

2. Comprovarem que realment tenim el repositori instal·lat i activat (per defecte ve activat), en aquest cas ens hauria d'apareixér el MySQL 8.0 Community.
```
# sudo yum repolist enabled | grep "mysql.*-community.*"
```
<details open>
<summary><b>Veure els repositoris instal·lats prèviament.</b></summary>
<img src="captures/check_repo.png">
</details>

3. Si no ens aparegués res hauriem d'habilitar aquest repositori
```
# sudo dnf config-manager --enable mysql80-community
```

4. A partir dels sistemes RHEL8 en els sistemes operatius hi ha un mòdul activat per defecte de MySQL, on enmascara tots els paquets de MySQL, llavors haurem de deshabilitar aquest mòdul per fer visibles els paquets del repositori de MySQL.

```
# sudo yum module disable mysql -y
```
<details open>
<summary><b>Deshabilitar el mòdul MySQL per defecte (Posterior a RHEL8.0).</b></summary>
<img src="captures/disable_default_module.png">
</details>

5. 




