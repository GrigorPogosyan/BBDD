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

4. A partir dels sistemes RHEL8 en posterior hi ha un mòdul activat per defecte de MySQL, on enmascara tots els paquets de MySQL, llavors haurem de deshabilitar aquest mòdul per fer visibles els paquets del repositori de MySQL.

```
# sudo yum module disable mysql -y
```
<details open>
<summary><b>Deshabilitar el mòdul MySQL per defecte (Posterior a RHEL8.0).</b></summary>
<img src="captures/disable_default_module.png">
</details>

5. I ara sí que podem instal·lar el MySQL amb les seves dependències en el nostre sistema.
```
# sudo yum install mysql-community-server -y
```
<details open>
<summary><b>Instal·lació MySQL</b></summary>
<img src="captures/install_mysql.png">
</details>

<details open>
<summary><b>Instal·lació Feta Correctament</b></summary>
<img src="captures/install_mysql2.png">
</details>

<hr>

## Post Instal·lació
Al acabar la instal·lació cal que fem un seguit de passos per acabar de tenir correctament configurat i montat el nostre MySQL Server 8.0.

1. Primer de tot encendrem el nostre servei de MySQL i comprovarem el seu estat, ha d'estar en <span style="color:green">active running</span>.
```
# sudo systemctl start mysqld

# sudo systemctl status mysqld
```
<details open>
<summary><b>Iniciem el servei Mysqld</b></summary>
<img src="captures/start_mysqld.png">
</details>

<details open>
<summary><b>Comprovem l'estat del servei.</b></summary>
<img src="captures/status_mysqld.png">
</details>

2. Per defecte al instal·lar el MySQL, en els logs s'ha generat la contrasenya del Root per poder accedir-hi, llavors filtrarem aquest apartat dels logs per veure la contrasenya.
```
# sudo grep 'temporary password' /var/log/mysqld.log
```
<details open>
<summary><b>Obtenir la contrasenya Root dels Logs.</b></summary>
<img src="captures/grep_root_password.png">
</details>

3. Ara executarem el script <span style="color:yellow">mysql_secure_installation</span>, millorar la seguretat del nostre MySQL instal·lat on haurem de configurar els següents apartats.

    > - Canvia la contrasenya del Root.
    > - Treure el accés remot per l'usuari Root.
    > - Treure els usuaris anonims.
    > - Treure la base de dades test on per defect poden accedir tots els usuaris i els anonims, i també reiniciar els privilegis de les bases de dades que comencin per "test_.".

    Executarem el mysql_secure_installation.
    ```
    # sudo mysql_secure_installation
    ```
    I configurarem les opcions que venen a continuació.
    <details open>
    <summary><b>Opció 1:  Canviar Contrasenya Root.</b></summary>
    <img src="captures/mysql_secure1.png">
    </details>

    <details open>
    <summary><b>Opció 2: Eliminar usuaris anonims.</b></summary>
    <img src="captures/mysql_secure1.png">
    </details>
    
    








