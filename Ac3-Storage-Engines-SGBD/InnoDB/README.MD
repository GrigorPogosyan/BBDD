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

