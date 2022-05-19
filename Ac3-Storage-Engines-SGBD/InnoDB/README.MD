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
En resum té aquesta mida perquè està definida en la variable, i funciona d'aquesta manera, per no començar a reservar 10000M de cop quan realment no ens fa falta. Per això està el autoextend que és el que farà que el nostre fitxer augmenti de mida.

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
Però si ens fixem bé, ells canviem la variable global del Storage Engine per defecte a MyIsam abans d'executar la taula on no té definida cap Storage Engine, i encara nosaltres tinguem definits a nivell de configuració que per defecte és InnoDB, en el fitxer es modifica aquesta variable per MyISAM, llavors hem de treure aquesta linea o especificar el Storage Engine InnoDB en aquesta taula.

Millor definir-lo en la sentència DDL.

```
Engine = InnoDB
```

<details open>
<summary><b>Com es veuria abans d'especificar InnoDB</b></summary>
<img src="captures/abans_posar.png">
</details>
<details open>
<summary><b>Com es veuria després d'especificar InnoDB</b></summary>
<img src="captures/despres_posar.png">
</details>

Ara quan sabem que tenim totes les taules amb el Storage Engine InnoDB, procedirem a importar-les en el nostre MySQL.

Entrarem a MySQL en local, i importarem el Schema. I les dades del Schema
```
mysql> SOURCE /root/sakila-db/sakila-schema.sql
```
<details open>
<summary><b>Importar BD Sakila (Schema)</b></summary>
<img src="captures/import_sakila_schema.png">
</details>

```
mysql> SOURCE /root/sakila-db/sakila-data.sql
```
<details open>
<summary><b>Importar BD Sakila (Data)</b></summary>
<img src="captures/import_sakila_data.png">
</details>

I ja ho tindriem importat

## Fitxers de dades, Ruta i Mida
Ara, com tenim el paràmetre `innodb_file_per_table` desactivat, ja no crea fitxers per cada taula de BD, ara les crea tot en un únic fitxer juntament amb els altres BD. 
Antigament, si anessim a `/var/lib/mysql/sakila` veuriem els diferents fitxers en format .ibd de les diferents taules, però ara estarà buit aquesta carpeta i tot ho tindrem en el fitxer ibdata1 en binari, on la seva ruta completa seria `/var/lib/mysql/ibdata1`.

Si mirem la ruta `/var/lib/mysql/sakila` podrem veure que està buida.
```
mysql> SOURCE /root/sakila-db/sakila-data.sql
```
<details open>
<summary><b>Verificar que ja no crea els .ibd</b></summary>
<img src="captures/sakila_buit.png">
</details>

I si mirem ara el fitxer ibdata1 que és on es guarda tota la informació de totes les bases de dades podrem veure que ha crescut, i el autoextend li ha sumat els `64M` que tenia configurat per defecte. Ara el fitxer té `76M` reservats, això vol dir que ha superat els `12M` inicials.

<details open>
<summary><b>Veure que ha crescut la mida del nostre System TableSpace ibdata1 degut a les importacions de dades anteriors</b></summary>
<img src="captures/alish_ibda.png">
</details>

## Canviar la configuració de dades de MySQL
En aquest apartat farem els següents canvis en el nostre MySQL.

> - Canviar la localització del directori de dades.
>- Tenir 2 fitxers de System Tablespace complint lo següent:
>   - Tots 2 fitxers amb 50MB de mida inicial.
>   - El tablespace ha de créixer de 5MB en 5MB
>       - Situarem aquests fitxers en un disc diferent cadascun que insertarem i montarem en la nostra màquina vitual

### Canviar Localització del directori de Dades
En aquest punt canviarem la localització de dades del MySQL a la ruta `/hd-mysql`, és a dir a l'arrel. Ho farem sense passar els arxius de l'antiga ruta, farem que ho torni a generar ell.

Llavors crearem el directori.
```
# mkdir /hd-mysql/
```
<details open>
<summary><b>Crear el directori</b></summary>
<img src="captures/mkdir_hdmysql.png">
</details>

Ara, hem de posar els permisos corresponents al directori per a que MySQL pugui accedir correctament i no tingui cap problema. Els permisos chmod 751, volen dir RWX al propietari RX al grup i X als altres.

```
# chown -R mysql:mysql /hd-mysql
# chmod 751 /hd-mysql
```
<details open>
<summary><b>Permisos del nou directori del MySQL</b></summary>
<img src="captures/permisos_hd-mysql.png">
</details>

Ara, procedirem a canviar el directori de dades del MySQL des del fitxer de configuració `/etc/my.cnf`. Haurem de tocar els paràmetres `datadir` i `datadir` i deixar els paràmetres `log-error` i `pid-file` tal com ens venen per defecte. Aquests paràmetres els posarem a sota del `[mysqld]`. Els paràmetres que fan referència per exemple a MyRocks que l'hem hagut d'instal·lar, les hem de treure perquè al canviar de directori i generar nous fitxers de configuració les instal·lacions externes que hem fet per exemple del MyRocks els perdriem. 

També, en la primera vegada que hem de reiniciar el servei del MySQL per a que ens generi tota l'estructura de fitxers, bd etc... Hem de tenir el my.cnf pelat, és a dir totes les variables que hem configurat de tipus: `innodb_file_per_table`, `default-storage-engine=InnoDB` les hem de comentar perquè sinó ens donarà error al reiniciar el servei, sempre al primer cop quan ha de crear l'estructura de fitxers al reiniciar el servei només hem de tenir sense comentar les variables `datadir`,`datadir`,`log-error`,`pid-file`. Després de que es reinici el servei i es crei l'estructura de fitxers, ja podem descomentar les linies anteriors que teniem. Però compte perquè tota la configuració ja sigui Storage Engine o un altre configuració instal·lat externament haurem de tornar-les a instal·lar si les volem tenir, perquè al generar-se nous fitxers de configuració aquestes externes es perden.

Llavors obrirem el fitxer de configuració de MySQL
```
# nano /etc/my.cnf
```
I borrarem / comentarem totes les lineas de configuració de variables que tenen a veure amb característiques instal·lades externs, en aquest cas de MyRocks, perquè al generar-se un nou directori de dades sense fer còpia de l'anterior, totes les instal·lacions externes es perden i no té sentit intentar configurar una variable d'una característica (Storage Engine en aquest cas però podria ser un altre) que no en tenim instal·lada.

També comentarem les lineas de configuració (si en tenim), d'altres paràmetres de MySQL, com per exemple el `default-storage-engine`, `innodb_file_per_table` perquè en el primer reinici que farem més endavant hem de tenir només especificat `datadir`, `socket`, `pid-file (el deixarem per defecte)` i `log-error (el deixarem per defecte)`
```
[mysqld]
datadir=/hd-mysql
socket=/hd-mysql/mysql.sock
```
<details open>
<summary><b>Comentar les lineas sobrants i canviar el directori de dades i socket</b></summary>
<img src="captures/mycnf-datadir.png">
</details>

Guardem el fitxer de configuració i sortim.

Ara, si estem utilitzant RHEL / CentOS , si no tenim el `SELINUX` desactivat hem de afegir el `SELinux Security Context` a la nova ubicació, per a que funcioni. També el podrem deshabilitar per evitar problemes
```
# semanage fcontext -a -t mysqld_db_t "/hd-mysql(/.*)?"
# restorecon -R /hd-mysql
```
<details open>
<summary><b>Afegir el SELinux Security Context a la nova ubicació</b></summary>
<img src="captures/selinux.png">
</details>

Un cop fet això, si reiniciem el servei de MySQL i mirem l'estatus no hi hauriem de tenir cap problema.
```
# systemctl restart mysqld && systemctl status mysqld
```
<details open>
<summary><b>Reiniciar i veure l'estat del Servei</b></summary>
<img src="captures/restart_status.png">
</details>

I si mirem en el nou directori, podrem veure com ha reproduït tota la estructura.

⚠️ ATENCIÓ ⚠️ : SI FEM AQUEST PAS DE REINICIAR, DESPRÉS HEM DE TORNAR A BORRAR TOT I CREAT TOT DE NOU PERQUÈ EN L'APARTAT DE IBDATA CANVIAREM EL DIRECTORI I NO MANTINDREM EL IBDATA1 QUE ENS CREARÀ ARA A `/HD-MYSQL`. LLAVORS TORNAREM A GENERAR LA ESTRUCTURA PERÒ LOCALITZANT ELS IBDATA EN UN LLOC DIFERENT. LLAVORS SI NO CANVIARÀS ELS PATH DEL IBDATA POTS FER AQUEST PAS TRANQUILAMENT.

<details open>
<summary><b>Comprovar que ha creat els fitxers (tota la estructura)</b></summary>
<img src="captures/lshd-mysql.png">
</details>

Ara, tornarem al `my.cnf`, i descomentarem les linies comentades anteriorment, on modificavem algunes variables del MySQL, excepte les variables que fan referència a una característica instal·lada, que ara al ser nova estructura no el tenim i per tant l'hem d'instal·lar de nou si el volem fer servir.

Llavors descomentarem les linies (si en teniem abans o volem tenir) i també afegirem una configuració extra, per a que ens funcioni el login per MySQL i no ens doni error de `Socket` més endavant, perquè aquestruta del Socket si no l'especifiques en la secció de `[Client]` utilitza la que té per defecte, i com hem canviat de directori hem d'especificar el del nou directori.
```
# nano /etc/my.cnf
```
```
[client]
socket=/hd-mysql/mysql.sock
```
<details open>
<summary><b>Descomentar les linies de variables que utilitzem, i configurar per a que funcioni el socket per part del client</b></summary>
<img src="captures/mycnf2.png">
</details>

Guardarem i reiniciarem el Servei.
```
# systemctl restart mysqld && systemctl status mysqld
```
<details open>
<summary><b>Reiniciar i veure l'estat del Servei</b></summary>
<img src="captures/restart_status2.png">
</details>

Ara, hem d'anar a l'arxiu de logs i buscar la contrasenya que ens haurà generat del Root
```
# cat /var/log/mysql.log | grep "Generated"
```
<details open>
<summary><b>Obtenir la contrasenya del Root</b></summary>
<img src="captures/password_generated.png">
</details>

Ara, iniciarem sessió amb root i canviarem la contrasenya.
```
# mysql -u root -p"pwd_generat"
```
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'N3wP4ssw0rd'; flush privileges;
```
<details open>
<summary><b>Iniciar sessió i canviar contrasenya</b></summary>
<img src="captures/change_pwd.png">
</details>

Si volem, també podem comprovar que s'ha canviat mirant les variables pel MySQL.
```
mysql> SHOW VARIABLES WHERE Value LIKE '%/hd%';
```
<details open>
<summary><b>Comprovar nova ruta de les variables</b></summary>
<img src="captures/valuelikehd.png">
</details>

Ara, per últim lloc, si ens volem assegurar que el nostre MySQL ja no depén dels fitxers de l'antiga ruta podem renombrar l'antiga ruta i reiniciar el MySQL, si no dona error és que ho tenim bé i hem canviat la localització correctament.

Després tornarem a posar el nom antic per si volem tornar a la ruta.
<details open>
<summary><b>Comprovar que el nostre MySQL ja no depén de l'antiga ruta</b></summary>
<img src="captures/comprobacion.png">
</details>

### Configuració System Tablespace en 2 Discos
En aquest apartat, configurarem 2 System TablesSpace. Els dos amb mida 50MB inicial, fent-los creixent de 5 MB en 5 MB i el `ibdata1` estàra en `/disk1/` i el `ibdata2` estarà en `/disk2/`.

Llavors començem insertant els discos virtuals en la nostra màquina virtual / real si estem fent-ho en un cas real.

Els discos en aquest cas de prova són de 2GB cadascún.
<details open>
<summary><b>Disk 1 creat i insertat</b></summary>
<img src="captures/new_hard_disk1.png">
</details>
<details open>
<summary><b>Disk 2 creat i insertat</b></summary>
<img src="captures/new_hard_disk2.png">
</details>

Ara iniciarem la màquina.
Comprovarem que tenim més discos insertats amb la comanda lsblk
```
# lsblk
```
<details open>
<summary><b>Veure que el sistema reconeix els 2 discos</b></summary>
<img src="captures/lsblk.png">
</details>

Ara accedirem al menu del programa `fdisk` indicant el disk1, és a dir que els canvis realitzats dins d'aquest menú es realitzaràn sobre el disk1

```
# sudo fdisk /dev/sdb
``` 
Després premem la lletra `M` i `Intro` per veure les opcions que tenim amb el disk
<details open>
<summary><b>Accedir al menu fdisk indicant el disc /dev/sdb (Disk1)</b></summary>
<img src="captures/menufdisk.png">
</details>

Ara, crearem una partició primaria.
<details open>
<summary><b>Crear partició primària</b></summary>
<img src="captures/partitionprimary.png">
</details>

Mirarem com ha quedat la particií i després, guardarem els canvis realitzats i sortirem del menu.
<details open>
<summary><b>Veurem els canvis com han quedat, els guardarem i sortirem.</b></summary>
<img src="captures/writechanges.png">
</details>

Ara, hem de formatar la partició en el sistema de fitxers de linux `ext4`. Com abans hem consultat la partició sabem com es diu, en aquest cas `dev/sdb1`
```
# sudo mkfs -t ex4 /dev/sdb1
```
<details open>
<summary><b>Formatar la partició a ext4</b></summary>
<img src="captures/mkfsdevsdb1.png">
</details>

I ara si consultem la partició podem veure que ja està preparat per ser montat
```
# lsblk -o NAME,FSTYPE,SIZE /dev/sdb1
```
<details open>
<summary><b>Veure com ha quedat la partició del primer disc</b></summary>
<img src="captures/lsblkparam.png">
</details>

Informació de creació de particions extreta [d'aquesta pàgina](https://www.sapalomera.cat/moodlecf/apunts/smx/sox/index.html?cap=100&ref=2232).

**LLAVORS FARIEM LO MATEIX PERÒ PER L'ALTRE DISC (DISC2)**

I així es veuria el segon disc al fer els passos anteriors.
<details open>
<summary><b>Veure com ha quedat la partició del segon disc</b></summary>
<img src="captures/lsblkparam2.png">
</details>

Arribat en aquest punt tenim preparat lo següent:

- Disc 1 Partició 1 --> `/dev/sdb1` fs=ext4
- Disc 2 Partició 1 --> `/dev/sdc1` fs=ext4

Ara, crearem els directoris de /disk1 i /disk2 a l'arrel que és on muntarem els discs amb els permisos adients per el MySQL. Encara que després de montar el disc sobre la carpeta pot ser que els permisos configurats ara desapareixin i es posin els de root
```
# mkdir /disk1
# chown -R mysql:mysql /disk1
# chmod 751 /disk1

# mkdir /disk2
# chown -R mysql:mysql /disk2
# chmod 751 /disk2
```
<details open>
<summary><b>Creació de carpetes i permisos on muntarem els discs</b></summary>
<img src="captures/mkdir_disk1_disk2.png">
</details>

Ara, muntarem les particions temporalment en el seu directori corresponent per veure si hi ha algún problema i veure com ho monta.
```
# sudo mount -t ext4 /dev/sdb1 /disk1
# sudo mount -t ext4 /dev/sdc1 /disk2
# df -lh
```
<details open>
<summary><b>Muntar les particions manualment i veure si hi ha algun problema</b></summary>
<img src="captures/temporal_mount.png">
</details>


Un cop sabem que pot muntar-ho sense problema, en comptes de muntar-ho manualment afegirem unes linies en `etc/fstab` per a que cada vegada que s'inicii el sistema operatiu ell ho monti automàticament i no tinguem cap problema.

```
# nano /etc/fstab
```

I afegirem les següents lineas per a que ho monti.
```
# Muntar /dev/sdb1 en /disk1
/dev/sdb1   /disk1     ext4    defaults        0       3

# Muntar /dev/sdc1 en /disk2
/dev/sdc1   /disk2     ext4    defaults        0       3
```
<details open>
<summary><b>Montar les particions en els directoris des d'el Fstab de manera que al iniciar el sistema es montin automàticament</b></summary>
<img src="captures/fstab.png">
</details>

Ara guardarem i reiniciarem la màquina.
<details open>
<summary><b>Reiniciar la màquina</b></summary>
<img src="captures/init6.png">
</details>

Un cop que s'inicii amb la comanda `mount` mirarem si ens ha montat el disc i després amb la comanda df -lh ho tornarem a verificar, així estarem segurs al 100%.
```
# mount | grep "disk"
# df -lh
```
<details open>
<summary><b>Comprobació de que s'ha montat automàticament correctament</b></summary>
<img src="captures/mount_dflh.png">
</details>

Ara, al montar el disc sobre la carpeta, pot ser que es canviin els permisos a root automàticament.
Si passa això tornem a posar els permisos, propietari i grup d'aquestes carpetes.
```
# chown -R mysql:mysql /disk1
# chmod 751 /disk1

# chown -R mysql:mysql /disk2
# chmod 751 /disk2
```
<details open>
<summary><b>Permisos Discs</b></summary>
<img src="captures/permisos_disk.png">
</details>

Arribat en aquest punt ja tenim preparat els discs, el directori amb els permisos adients.

⚠️ATENCIÓ⚠️: EN AQUEST PAS CANVIAREM LA RUTA DELS IBDATA SENSE MANTENIR L'ANTERIOR QUE ENS HA GENERAT, AIXÒ VOL DIR RE GENERAREM LA ESTRUCTURA DEL `/hd-mysql` LOCALITZANT EN UN LLOC DIFERENTS ELS IBDATA I PERDREM LES DADES ANTERIORS, SI VOLEM QUE LES DADES ANTIGUES ES MANTINGUIN HAUREM D'ESPECIFICAR 3 IBDATA, L'ANTIC, EL NOU I EL SEGON NOU PERÒ EN AQUEST CAS FAREM LA VERSIÓ DE RE GENERAR-LOS.

Ara el que farem serà canviar la ruta dels `ibdata` cap a discos externs.

Primer, desactivarem el `SE LINUX` si estem utilitzant un SO `RHEL` o `CENTOS`, podriem afegir la regla a la nova ubicació, però per evitar més problemes el desactivem.
```
# nano /etc/selinux/config
SELINUX=disabled
```
<details open>
<summary><b>Desactivar SELINUX</b></summary>
<img src="captures/disable_selinux.png">
</details>

Posteriorment, guardarem els canvis i reiniciarem la màquina.

```
# init 6
```

Com s'ha dit anteriorment, com regenerarem el contingut i perdrem les dades, hem de buidar el contingut de `/hd-mysql` o la ruta del destí que tinguem per a que torni a generar tots els fitxers, si canviem de destí no fa falta aquest pas.

```
# rm -r -R -f /hd-mysql/*
```

<details open>
<summary><b>Borrar el contingut de la carpeta destí per generar els fitxers</b></summary>
<img src="captures/vaciar_destino.png">
</details>


Anirem a l'arxiu de configuració de MySQL situat a `/etc/my.cnf`.
```
# nano /etc/my.cnf
```
I afegirem les següents linies en la secció de `[mysqld]`.

La següent linia treiem el path del data_home_dir, perquè després en el ibdata especificarem la ruta absoluta on guardarem els ibdata. Sinó concatenara la ruta d'aquest paràmetres més la ruta que es posi en el ibdata.
- `innodb_data_home_dir =`  

La següent linia especificarem la ruta absoluta de cada ibdata, amb la mida inicial de 50M, i el segon ibdata creixerà automàticament a mesura que ho necessiti. Si en el paràmetre anterior especifiquem alguna ruta, utilitzarà la ruta anterior + la que es posi en aquest paràmetre
`
- `innodb_data_file_path=/disk1/ibdata1:50M;/disk2/ibdata2:50M:autoextend`

La següent linia indicarà de quina mida anirà creixent el ibdata que tingui autoextend.
- `innodb_autoextend_increment = 5`

```
[mysqld]
datadir=/hd-mysql
socket=/hd-mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

innodb_file_per_table=OFF

innodb_data_home_dir = 
innodb_data_file_path=/disk1/ibdata1:50M;/disk2/ibdata2:50M:autoextend
innodb_autoextend_increment=5

mysqlx_socket = /hd-mysql/mysqlx.sock

[client]
socket=/hd-mysql/mysql.sock
```

També si mirem en els apartats anterior s'explica que en el fitxer de configuracio `my.cnf` en la secció de `[mysqld]` haviem de comentar els paràmetres opcionals que teniem posats per a que la primera vegada que regenerra l'estructura no doni error, per exemple el paràmetre de `default-storage-engine` s'ha de comentar (després de que monti la estructura podem descomentar aquestes linees). També posarem la secció de `[client]` especificant la nova ruta del `Socket` per a que ens deixi loguejar-nos.
També podem especificar el socket del `Plugin X` de MySQL, que es diu `mysqlx.sock`.

<details open>
<summary><b>Configuració per regenerar la estructura i tenir 2 System Tablespace en llocs diferents</b></summary>
<img src="captures/innodb_data.png">
</details>

Ara, reinicirem el servei de MySQL i si tot va bé haurà reiniciat amb èxit. Si no funciona amb éxit, borra els fitxers que ha generat al directori de dades, i tornarem a provar de reiniciar.

```
# systemctl restart mysqld
# systemctl status mysqld
```
<details open>
<summary><b>Reiniciar servei MySQL</b></summary>
<img src="captures/systemctl_status.png">
</details>

Si mirem el directori de dades podrem veure que ha generat tot.
```
# ls -alis /hd-mysql
```
<details open>
<summary><b>Directori de dades generat</b></summary>
<img src="captures/ls_alis_hdmysql.png">
</details>

I si mirem els discos en cada disc ha creat el seu ibdata amb `50MB` inicials.
```
# ls -alish /disk1
# ls -alish /disk2
```
<details open>
<summary><b>Veure els ibdata generat i la seva mida</b></summary>
<img src="captures/ls_alish_disk.png">
</details>

Ara, com s'ha fet en l'apartat de canviar el directori de MySQL, hem de tornar a buscar la contrasenya del Root en els logs, i tornar-la a canviar.
```
# cat /var/log/mysqld.log | grep "generated"
```
```
# mysql -u root -p"passwordgenerated"
```
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'N3WP4SSW0RD';
mysql> FLUSH PRIVILEGES;
```
<details open>
<summary><b>Canviar contrasenya del Root</b></summary>
<img src="captures/recover_password.png">
</details>

I ara també podem mirar per les variables del MySQL les rutes dels paràmetres que hem canviat.
```
mysql> SHOW VARIABLES WHERE VALUE LIKE '%/hd-mysql%';
mysql> SHOW VARIABLES WHERE VALUE LIKE '%/dis%';
mysql> SHOW VARIABLES LIKE '%autoextend%';
mysql> SHOW VARIABLES LIKE '%innodb_data_h%';
```
<details open>
<summary><b>Veure que han canviat els path dels variables com a comprovació</b></summary>
<img src="captures/show_variables_change.png">
</details>

Ara, tornarem a importar la BD de Sakila el Schema com a taules InnoDB, (ens assegurarem que tenim per defecte el Storage Engine InnoDB). 
```
mysql> SOURCE /root/sakila-db/sakila-schema.sql;
mysql> SOURCE /root/sakila-db/sakila-data.sql;
```
<details open>
<summary><b>Importar BD Sakila (Schema)</b></summary>
<img src="captures/import_sakila_schema.png">
</details>
<details open>
<summary><b>Importar BD Sakila (Data)</b></summary>
<img src="captures/import_sakila_data.png">
</details>
<hr>

## Tablespace per cada taula en una ruta especificada.
Partint del punt anterior, modificarem la configuració del MySQL, per a que cada taula generi el seu propi tablespace dins d'una carpeta que crearem preparada per aquests tablespaces.

Llavors començarem creant la carpeta que es dirà `tspaces` i estarà situada a `/tspaces`, la ruta d'aquest arxiu serà `/tspaces`, hem de crear la carpeta de tspaces en un lloc diferent del directori de dades, perquè per raons de seguretat no ens deixarà posar el data directory de la taula a una ruta descendent del directori de dades.

Crearem la carpeta i li posarem els permisos adients de MySQL
```
# mkdir /tspaces
# chown mysql:mysql -R /tspaces
# chmod 751 -R /tspaces
```
<details open>
<summary><b>Crear i preparar el directori dels tablespaces</b></summary>
<img src="captures/mkdir_tspaces.png">
</details>

🟢 MANERA 1🟢

Anirem al `my.cnf` i posarem en ON o treiem la variable de `innodb_file_per_table` d'aquesta manera cada taula generarà el seu propi tablespace, per defecte si no hi ha aquest paràmetre especificat en el `my.cnf`, genera el seu propi tablespace. També posarem el paràmetre `innodb_directories` especificant el directori de confiança `/tspaces`, perquè si tenim un directori apart creat per nosaltres i no és natiu del MySQL, tindrem problemes, l'haurem de dir en el paràmetre que aquest directori forma part dels directoris d'InnoDB
```
# nano /etc/my.cnf

[mysqld]
innodb_file_per_table=ON
innodb_directories="/tspaces";
```
<details open>
<summary><b>Activar el paràmetre innodb_file_per_table i especificarem innodb_directories</b></summary>
<img src="captures/innodb_file_per_table_directories.png">
</details>

Posteriorment reiniciarem i comprovarem el servei de MySQLD
```
# systemctl restart mysqld && systemctl status mysqld
```
<details open>
<summary><b>Reiniciar i comprovar el servei de MySQL</b></summary>
<img src="captures/systemctl_status2.png">
</details>

Si volem que per defecte els tablespaces de les taules es generin en un data directori diferent a la del `datadir`, no podem fer-ho, no hi ha cap opció. El que si que podem fer és alhora de crear la taula especificar en la mateixa sentència el data directori d'aquella taula i d'aquesta manera canviar-li la ruta del tablespace que generarà aquesta taula. Perquè per el tablespace de les taules les genera en el directori `datadir` del `MySQL` dins d'una carpeta de la seva BD.

Llavors el que farem, serà en la sentència DDL especificaR amb el paràmetre `DATA DIRECTORY` la ruta on volem que generi el tablespace de la taula

Llavors ara, entrarem al MySQL i crearem nostra BD de prova amb la taula especificant el directori del tablespace creat anteriorment.
```
mysql>CREATE DATABASE provaSpace;
mysql>USE provaSpace;
mysql>CREATE TABLE taulaProves (id INT PRIMARY KEY, nom VARCHAR(30)) DATA DIRECTORY="/tspaces/";
```
<details open>
<summary><b>Crear taula especificant el directori del tablespace</b></summary>
<img src="captures/data_directory_tspaces.png">
</details>

🟢 MANERA 2 🟢

També tenim una manera alternativa de crear el tablespace d'una taula en un directori especícic. Aquesta manera alternativa, és per si no volem especificar els paràmetres `innodb_file_per_table` i `innodb_directories` en el `my.cnf` i ho volem especificar només en la sentència DDL de la creació de taula, fent d'aquesta manera una "creació excepcional de taula"

És a dir en la sentència DDL posant uns paràmetres podem fer una creació excepcional i dir que aquesta taula tindrà el innodb_file_per_table i el seu directori de dades estarà situada a X lloc, sense la necessitat de modificar paràmetres a nivell de fitxer.
```
mysql>CREATE DATABASE provaSpace;
mysql>USE provaSpace;
mysql>CREATE TABLE taulaProves (id INT PRIMARY KEY, nom VARCHAR(30)) TABLESPACE=innodb_file_per_table DATA DIRECTORY="/tspaces/";
```
En la comanda anterior, especificarem que el `tablespace` sigui `innodb_file_per_table`, segons les fonts de MySQL amb aquest paràmetre t'estalvies activar el paràmetre per el `my.cnf`, i el paràmetre `DATA DIRECTORY` tindrà com a valor la ruta on volem guardar el tablespace..

I d'aquesta manera creariem el tablespace de la taula en una ruta diferent sense tocar paràmetres a nivell de fitxer

<details open>
<summary><b>Creació de la taula especificant el directori del tablespace</b></summary>
<img src="captures/tablespace-datadirectory.png">
</details>

⚠️Atenció: Si es posa el paràmetre `DATA DIRECTORY` sense que acompanyi el paràmetre `TABLESPACE` (encara que estigui ON a nivell de fitxer), posiblement doni un error, la solució del error en molts casos és posant el paràmetre `TABLESPACE`, o si ho volem fer de la manera 1, especificar el `innodb_directories`

<details open>
<summary><b>Posible error i solució</b></summary>
<img src="captures/tablespace-datadirectoryerror.png">
</details>

🟢 COMPROVACIONS 🟢

I si mirem la ruta del DATA DIRECTORY que hem especificat abans podem trobat el tablespace de la taula. Cada taula estarà dins d'una carpeta que es dirà amb el nom de la BD que pertany la taula, funcionarà qualsevol de les dues maneres
```
# ls -alish /tspaces/provaSpace/
```
<details open>
<summary><b>Verificar que guarda el tablespace en la carpeta tspaces</b></summary>
<img src="captures/ls-tspaces-provaSpace.png">
</details>

I mirarem també com ha quedat la carpeta de la BD de la taula en el directori de dades que en teoria ha de quedar buida perquè la única taula que té l'hem creat en una ruta diferent.
```
# ls -alish /hd-mysql/provaSpace
```
<details open>
<summary><b>Verificar que la carpeta de la BD del directori de dades anterior ha quedat buida</b></summary>
<img src="captures/ls-provaSpace-vacio.png">
</details>
<hr>

## Creació Tablespace Generals
En aquest apartat crearem tablespaces generals en discos diferents i dividirem les taules de la BD de Sakila en aquests dos tablespaces situats en llocs diferents.

### Tablespace ts1 a /disk1 i ts2 a /disk2
Crearem un tablespace anomenat ts1 a `/disk1` on situarem les taules de "actor", "address", i "category" de la BD de Sakila.

Ara, la opció innodb_file_per_table ens es igual.

El que farem serà d'un principi anar a l'arxiu de configuració `my.cnf` i afegir "directoris de confiança del innodb" apuntant cap al destí per evitar un error al crear el tablespace fora del directori de dades.
```
# nano /etc/my.cnf
```
```
[mysqld]
innodb_directories=/disk1;/disk2
```
<details open>
<summary><b>Afegit "paths" de confiança en el innodb per poder guardar dades fora de la ruta del directori de dades</b></summary>
<img src="captures/innodb_directories.png">
</details>

Amb lo anterior ens estalviem el següent error alhora de crear el tablespace en un path diferent a la del directori de dades:
<details open>
<summary><b>Error que tindriem al crear el tablespace fora del directori de dades si no posem el paràmetre anterior.</b></summary>
<img src="captures/error_confianca.png">
</details>

Reiniciarem el servei i el comprovarem.
```
# systemctl restart mysqld && systemctl status mysqld
```
<details open>
<summary><b>Reiniciar el servei per aplicar els canvis anteriors</b></summary>
<img src="captures/systemctl_status3.png">
</details>

Si volem comprovar que la variable s'ha modificat ho podem veure mitjançant una consulta SQL.
```
mysql> SHOW VARIABLES LIKE '%directori%';
```
<details open>
<summary><b>Comprovar que la variable innodb_directories s'ha modificat correctament</b></summary>
<img src="captures/like_directori.png">
</details>

Posteriorment crearem els tablespaces `ts1` situat a `/disk1` i el `ts2` situat a `/disk2` mitjançant sentències SQL. (Tenim els permisos de carpeta de disk1 i disk2 configurats en els apartats anteriors)
```
mysql> CREATE TABLESPACE ts1 ADD DATAFILE '/disk1/ts1.ibd' ENGINE=InnoDB;
```
```
mysql> CREATE TABLESPACE ts2 ADD DATAFILE '/disk2/ts2.ibd' ENGINE=InnoDB;
```
<details open>
<summary><b>Creació dels tablespaces</b></summary>
<img src="captures/create_tablespace.png">
</details>

Podem llistar els directoris i podem mirar que s'han creat correctament.
```
# ls -alish /disk1
# ls -alish /disk2
```
<details open>
<summary><b>Veure que ha creat els tablespaces en els directoris</b></summary>
<img src="captures/ls_ts.png">
</details>

### Importar Sakila col·locant les taules en els tablespaces
Ara, importarem la BD de Sakila però les taules repartides en els tablespaces que hem creat anteriorment.

Si ja tenim la BD Sakila, haurem de modificar el tablespace mitjançant la sentència ALTER. Modificarem la taula i li assignarem el tablespace, pero a continuació farem la versió d'importació.

El que hem de fer ara és modificar les sentències DDL de les creacions de les taules `actor`,`address`, i `category` i assignar aquests en el tablespace `ts1`.

Llavors obrirem el Schema DB de Sakila per editar el contingut
```
# nano /root/sakila-db/sakila-schema.sql
```
I a les taules actor, address i category, afegirem el tablespace `ts1` creat anteriorment, de manera que es veurien així finalment les taules amb el paràmetre `TABLESPACE`, encara que estigui posat, important que les taules estiguin en InnoDB!:
```
TABLESPACE ts1
```
<details open>
<summary><b>Canviar el DDL de les taules actor address i category i afegir-los al tablespace ts1 (abans d'importar-lo a la BD)</b></summary>
<img src="captures/tablespace_ts1.png">
</details>

I a la resta de taules els afegirem al tablespace ts2
```
TABLESPACE ts2
```
<details open>
<summary><b>Canviar el DDL de la resta de taules i afegir-los al tablespace ts2 (abans d'importar-lo a la BD)</b></summary>
<img src="captures/tablespace_ts2.png">
<img src="captures/tablespace_ts2_1.png">
<img src="captures/tablespace_ts2_2.png">
<img src="captures/tablespace_ts2_3.png">
<img src="captures/tablespace_ts2_4.png">
<img src="captures/tablespace_ts2_5.png">
</details>

Guardarem els canvis de DDL fets al fitxer SQL del Schema de Sakila, i procedirem a importar, recordar que en cas de tenir-lo importat i voler canviar el tablespace ES POT !!!
[+ Informació](https://dev.mysql.com/doc/refman/5.7/en/general-tablespaces.html)

```
mysql> SOURCE /root/sakila-db/sakila-schema.sql
```
<details open>
<summary><b>Importat l'Schema de Sakila amb les modificacions en les taules DDL que hem fet</b></summary>
<img src="captures/import_sakila_to_ts.png">
</details>

Un cop importat, per comprovar que les taules anteriors pertanyen cadascú al seu general tablespace corresponent ho podrem verificar amb les següents sentències, on LIKE 'ts1' equival al nom del nostre tablespace
```
mysql> SELECT a.NAME AS space_name, b.NAME AS table_name
FROM INFORMATION_SCHEMA.INNODB_TABLESPACES a,
INFORMATION_SCHEMA.INNODB_TABLES b 
WHERE a.SPACE=b.SPACE AND a.NAME LIKE 'ts1';
```
<details open>
<summary><b>Verificar les taules pertanyents al tablespace ts1</b></summary>
<img src="captures/check_ts1.png">
</details>

Ara verificarem les taules del ts2.
```
mysql> SELECT a.NAME AS space_name, b.NAME AS table_name
FROM INFORMATION_SCHEMA.INNODB_TABLESPACES a,
INFORMATION_SCHEMA.INNODB_TABLES b 
WHERE a.SPACE=b.SPACE AND a.NAME LIKE 'ts2';
```
<details open>
<summary><b>Verificar les taules pertanyents al tablespace ts2</b></summary>
<img src="captures/check_ts2.png">
</details>

A més podrem verificar la mida dels generals tablespaces `ts1` i `ts2`, si ho comparem amb la anterior mida que es pot veure en els apartats anteriors d'aquest document, aquesta mida ha pujat degut a les importacions que hem fet.
(Òbviament la mida del ts2 és major ja que conté més taules amb més camps)
```
# ls -lh /disk1/ts1.ibd
# ls -lh /disk2/ts2.ibd
```
<details open>
<summary><b>Verificar que la mida dels tablespaces ha pujat</b></summary>
<img src="captures/check_ts_size.png">
</details>

També podem mirar el directori de dades de la BD, hauria d'estar buit.
<details open>
<summary><b>Verificar que les taules no es guarden al directori per defecte, sinó que estan en els tablespaces</b></summary>
<img src="captures/vacio_sakila.png">
</details>

Un cop verificat que tenim ben configurats els tablespaces, procedirem a importar el contingut de les taules.
```
mysql> SOURCE /root/sakila-db/sakila-data.sql
```
<details open>
<summary><b>Importar el contingut de les taules Sakila</b></summary>
<img src="captures/sakila_data.png">
</details>

Ara, tornarem a verificar la mida dels tablespaces i podem veure que encara han crescut més.
```
# ls -lh /disk1/ts1.ibd
# ls -lh /disk2/ts2.ibd
```
<details open>
<summary><b>Verificar que la mida dels tablespaces ha tornat a pujar</b></summary>
<img src="captures/check_ts_size2.png">
</details>

Ara, finalment ens queda comprovar que podem fer sentències DML sobre aquestes dades guardades en diferents tablespaces.

Farem un parell de sentències "chorras", un a cada taula situat a cada tablespace.
```
mysql> UPDATE actor SET first_name = "bb" WHERE first_name RLIKE '.mp.';
```
<details open>
<summary><b>Canvis DML a la taula situada a ts1</b></summary>
<img src="captures/dml_ts1.png">
</details>

I comprovarem que s'han realitat els canvis
```
mysql> SELECT * FROM actor;
```
<details open>
<summary><b>Comprovació dels canvis realitzats de la taula actor</b></summary>
<img src="captures/dml_ts1_prueba.png">
</details>

Ara modificarem una taula que estigui en el ts2 com per exemple la taula `country`
```
mysql> UPDATE country SET last_update = now();
```
<details open>
<summary><b>Canvis DML a la taula situada a ts2</b></summary>
<img src="captures/dml_ts2_prueba.png">
</details>

<details open>
<summary><b>Comprovació dels canvis realitzats de la taula country</b></summary>
<img src="captures/dml_ts1_select.png">
</details>

Arribat en aquest punt ja sabem que tenim els tablespaces correctament configurats. 