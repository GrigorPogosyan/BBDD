# InnoDB<img align="right" width="200" src="../imatges/innodb_logo.png"/>
## Connexió per SSH al Sistema
Primer de tot, ens connectarem a la màquina per **SSH** desde CMD (ho tenim prèviament instal·lat).
```
ssh machineuser@ip 
```
<details open>
<summary><b>Connexió per SSH</b></summary>
<img src="captures/ssh.png">
</details>

<hr>

## Desactivar opció  innodb_file_per_table
Aquesta opció en els SGBD actuals ve per defecte activat, i la seva funció és que per cada taula creada de la meva base de dades tindrà el seu propi Table Space, és a dir cada taula tindrà el seu propi fitxer de dades que aquest conté tota la informació de la taula, son fitxers .ibd

Per desactivar aquesta opció i dir que ara no volem un fitxer per cada taula, sinó un fitxer per totes les taules haurem de desactivar aquesta opció.

Lo primer que hem de fer és anar al nostre fitxer de configuració de MySQL i afegir el següent paràmetre en OFF: `innodb_file_per_table=OFF`

```
# nano /etc/my.cnf
```
```
[mysqld]
innodb_file_per_table=OFF
```
<details open>
<summary><b>Desactivar innodb_file_per_table</b></summary>
<img src="captures/ssh.png">
</details>

Guardarem els canvis i reiniciarem el Servei de MySQLD.

```
# systemctl restart mysqld && systemctl status mysqld
```
<details open>
<summary><b>Reiniciar i veure l'estat del Servei</b></summary>
<img src="captures/restart_status.png">
</details>

Ara comprovarem que està desactivat consultant-ho a la variable des del MySQL.
```
mysql> SHOW VARIABLES LIKE '%file_per%';
```
<details open>
<summary><b>Verificar que l'hem desactivat</b></summary>
<img src="captures/status_file_per_table.png">
</details><br>

## Permisos, Usuari i Grup de la carpeta Datadir
Ara, veurem els permisos, l'usuari i el grup de la carpeta del directori de dades (datadir).

Si no sabem quin és el nostre directori de dades, també el podrem consultar via SQL.
```
mysql> SHOW VARIABLES LIKE '%datadir%';
```
<details open>
<summary><b>Consultar el nostre directori de dades</b></summary>
<img src="captures/show_datadir.png">
</details>

En aquest cas la ruta del path del directori de dades és `/var/lib/mysql`.

Per veure els permisos el usuari i el grup d'aquesta carpeta fariem lo següent:
```
# ls -alis /var/lib/mysql
```
<details open>
<summary><b>Veure permisos, usuari i grup del directori de dades</b></summary>
<img src="captures/ls_alis_var_lib_mysql.png">
</details>

Veient això podem arribar a la següent conclusió.

- Propietari del directori ➡️​ `mysql` (usuari)
- Grup del directori ➡️​ `mysql` (grup)
* Permisos:
    - Permisos per el propietari (`mysql`) ➡️​ RWX (lectura, escriptura i execució)
    - Permisos per el grup (`mysql`) ➡️​ RX (lectura i execució)
    - Altres ➡️​ X (Execució)
<br><br>

## Mida System TableSpace
Per veure la mida del nostre System TableSpace (fitxers ibdata), primer hem de saber on estan localitzats en el nostre SO. Per saber-ho, ho farem mitjançant una consulta SQL.
```
mysql> SHOW VARIABLES LIKE 'innodb_data_file_path';
```
<details open>
<summary><b>Saber la ruta per defecte del nostre System TableSpace i la mida inicial</b></summary>
<img src="captures/innodb_data_file_path.png">
</details>

El value anterior, indica que dins de la carpeta de MySQL, hi ha un fitxer que es diu `ibdata1`, i que de mida inicial té `12M`, reservarà aquests 12M inicials aquest arxiu.

També ho podem comprovar la mida des del directori.
```
# ls -lh /var/lib/mysql/ibdata1
```
<details open>
<summary><b>Mida del arxiu ibdata1</b></summary>
<img src="captures/ibdata1_size.png">
</details>

Té aquests 12M inicials, però quan s'omplin anirà, en aquests 12M se li sumarà la mida del valor del paràmetre `innodb_autoextend_increment`. Que per defecte son `64M`, llavors en total tindriem `76M` i en cas de que s'ompli aquesta mida, es tornaria a sumar a la mida `64M` que és el que tenim configurat en el paràmetre `innodb_autoextend_increment`, d'aquesta manera succesivament anirà creixent la mida del fitxer. I a més d'aquesta manera, estarem estalviant espai en el nostre disc, i no pas reservant des d'un principi 100M o el que pensem que ens faria falta.

```
mysql> SHOW VARIABLES LIKE 'innodb_autoextend_increment';
```
<details open>
<summary><b>Consultar la mida en Megabytes que se li sumarà a cada fitxer configurat amb autoextend.</b></summary>
<img src="captures/innodb_autoextend_increment.png">
</details>

## Importar BD Sakila com a taules InnoDB
Ara, importarem la BD Sakila que ens proporciona MySQL per fer proves com a taules InnoDB.

Tenim sort i MySQL ja ha definit l'engine a cada sentència de creació de taules i ens ha estalviat feina, no obstant això hi ha taula/les que no tenen definit cap Storage Engine i això vol dir que en les taules que no hi ha especificat el Storage Engine utilitzaria el Storage Engine per defecte que tenim configurat en el nostre SGBD.

Llavors el que farem serà posar InnoDB com a Storage Engine per defecte.
```
# nano /etc/my.cnf
```
```
[mysqld]
default-storage-engine=InnoDB
```
<details open>
<summary><b>InnoDB default Storage Engine</b></summary>
<img src="captures/innodb_default.png">
</details>

I seguit reiniciariem el servei de MySQL
```
# systemctl restart mysqld && systemctl status mysqld
```
<details open>
<summary><b>Reiniciar i veure l'estat del Servei</b></summary>
<img src="captures/restart_status.png">
</details>

Ara, ja tenim configurat InnoDB com al motor d'emmagatzematge per defecte, ara hem de descarregar-nos la BD Sakila [d'aquesta pàgina](https://dev.mysql.com/doc/index-other.html)

<details open>
<summary><b>Pàgina per descarregar Sakila</b></summary>
<img src="captures/sakila_page.png">
</details>

Nosaltres des de linux, farem un `wget` per descarregar-ho.
```
# wget https://downloads.mysql.com/docs/sakila-db.tar.gz
```
<details open>
<summary><b>Descarregar Sakila</b></summary>
<img src="captures/wget_sakila.png">
</details>

Ara, descomprimirem el .tar.gz obtingut de Sakila
```
# tar xzvf sakila-db.tar.gz
```
<details open>
<summary><b>Descomprimir Sakila</b></summary>
<img src="captures/tar_sakila.png">
</details>

Ara mirarem les sentències DDL sakila-schema.sql del Sakila que és el que ens interessa, per veure si hi ha alguna contradicció de Storage Engine, perquè volem que tot estigui en InnoDB.

Al fer la següent comprovació podem veure que una taula no té especificat el Storage Engine, (també podem mirar-ho manualment en el fitxer).
```
# cat sakila-db/sakila-schema.sql | grep "CREATE TABLE" -c
16
# cat sakila-db/sakila-schema.sql | grep "ENGINE=InnoDB" -c
15
```
<details open>
<summary><b>Comparació de sentències CREATE TABLE i veure si tenen especificat Engine</b></summary>
<img src="captures/grep_count.png">
</details>

Si revisem el fitxer manualment, podrem veure quina taula és. Si no especifica l'engine utilitzarà el que té per defecte el SGBD des del qual s'importa la BD.


