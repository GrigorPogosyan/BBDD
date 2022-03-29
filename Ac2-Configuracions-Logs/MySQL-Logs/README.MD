# Configuracions i comprovacions de logs en el MySQL (Percona Server8.0) <img align="right" width="70" src="../imatges/logs.ico"/>

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

## Tasques de Configuració i Comprovació de Logs

### Veure Logs Activats per Defecte

A MySQL treballarem els diferents 4 tipus de logs:

> - Logs d'Errors.
> - Log General de Consultes.
> - Log de Consultes Lentes.
> - Log de Binaris.
 
Per poder veure si venen activats per defecte o no, la manera més fàcil i ràpida és mitjançant sentències `SHOW VARIABLES LIKE '%NomLog%' ` (encara que també podem mirar-ho dintre de l'arxiu de configuració `/etc/my.cnf` però mitjançant sentències sabem que no ens deixarem cap, perquè hi ha configuracions que no venen en el /etc/my.cnf i venen per defecte activats). Llavors optarem la manera de saber-ho mitjançant sentències.

```
mysql> SHOW VARIABLES LIKE '%LOG_ERROR%';
```

<details open>
<summary><b>Log d'Errors: ACTIVAT sempre per DEFECTE </b></sumary>
<img src="captures/logerror.png">
</details>

```
mysql> SHOW VARIABLES LIKE '%GENERAL_LOG%';
```
<details open>
<summary><b>Log General: DESACTIVAT per DEFECTE </b></summary>
<img src="captures/generallog.png">
</details>

```
mysql> SHOW VARIABLES LIKE '%SLOW_QUERY_LOG%';
```
<details open>
<summary><b>Log de Consultes Lentes: DESACTIVAT per DEFECTE </b></summary>
<img src="captures/slowquerylog.png">
</details>

```
mysql> SHOW VARIABLES LIKE '%SLOW_QUERY_LOG%';
```
<details open>
<summary><b>Log de Consultes Lentes: DESACTIVAT per DEFECTE </b></summary>
<img src="captures/slowquerylog.png">
</details>

```
mysql> SHOW VARIABLES LIKE '%LOG_BIN%';
```
<details open>
<summary><b>Log de Binaris: ACTIVAT per DEFECTE (Depèn de la versió.)</b></summary>
<img src="captures/logbin.png">
</details>

## Activar els logs que no venen activats per Defecte en un fitxer de configuració de logs

Ara, crearem un fitxer de configuració de logs `/etc/percona/logs.cnf`, si no existeix la carpeta percona la creem. I ara dins del fitxer de configuració de logs que hem creat, només configurarem els que no vinguin activats per defecte, en aquest cas el Slow Query Log, i General Log, depenent de la versió també hauriem de configurar el Bin Log, però com es pot veure anteriorment ja ve activat per defecte.

1. Crearem la estructura de path on crearem aquest fitxer de logs

    ```
    # mkdir /etc/percona
    # touch logs.cnf
    ```

    <details open>
    <summary><b>Creació de la estructura de PATH</b></summary>
    <img src="captures/pathstructure.png">
    </details>

2. Obrirem el fitxer de configuració amb l'editor nano, i activarem els logs que no ens venen activats per defecte, en el meu cas el Log General de Consultes i el Log de Consultes Lentes, posarem el path que té per defecte /var/lib/mysql/ pero canviarem el nom del fitxer del log per a que sigui més intuïtiu la seva funció. El fitxer .log ho crearà ell mateix. En cas de que el Path del log que li posem, no tingui accéss el MySQL li hauriem de donar permisos.

    ```
    # nano /etc/percona/logs.cnf

    [mysqld]
    #GENERAL LOG AMB EL PATH QUE VE PER DEFECTE
    general_log = 1
    log_output = FILE
    general_log_file = /var/lib/mysql/general.log

    #LOG DE CONSULTES LENTES AMB EL PATH QUE VE PER DEFECTE I SI EL TEMPS D'EXECUCIÓ PASSA DE 2 SEGONS ES REGISTRA LA CONSULTA.
    slow_query_log = 1
    slow_query_log_file = /var/lib/mysql/slow-query.log
    long_query_time = 2
    ```
    <details open>
    <summary><b>Fitxer logs.cnf</b></summary>
    <img src="captures/logscnf.png">
    </details>

3. Un cop configurat el fitxer de logs, hem d'anar al fitxer de configuració de MySQL i des d'aquest fitxer cridar el fitxer creat de logs. D'aquesta manera, cada cop que s'inicia el sistema, es crida al my.cnf i des del my.cnf cridem al fitxer de configuració de logs afegirem la linea següent.

    ```
    # nano /etc/my.cnf
    [mysqld]
    !includedir /etc/percona
    ```

    <details open>
    <summary><b>Incloure directori de configuració de logs des del fitxer de configuració principal /etc/my.cnf</b></summary>
    <img src="captures/includedir.png">
    </details>

4. Reiniciarem el servei i comprovarem que no hi ha cap fallada.

    ```
    # systemctl restart mysqld

    # systemctl status mysqld
    ```
    <details open>
    <summary><b>Reiniciar i comprovar Servei</b></summary>
    <img src="captures/reiniciar.png">
    </details>

## Comprovació de la activació i funcionament dels logs.

Un cop activats els logs, que no teniem activats, ho podem verificar tornant a fer les sentències i consultant si està en OFF o en ON el log.

1. Accedirem al MySQL i comprovarem per cada log activat que estigui en ON.

    ```
    mysql> SHOW VARIABLES LIKE '%GENERAL_LOG%';

    mysql> SHOW VARIABLES LIKE '%SLOW_QUERY_LOG%';
    ```
    <details open>
    <summary><b>Comprovar que estan actius aquests logs</b></summary>
    <img src="captures/showvariable.png">
    </details>


2. També mirarem si ha creat aquests fitxers de log i que el propietari sigui mysql o que tingui permisos d'escriptura i lectura.

    ```
    # ls -alis /var/lib/mysql/*.log
    ```

    <details open>
    <summary><b>Mirar la creació dels logs i permisos</b></summary>
    <img src="captures/lslogs.png">
    </details>

3. Farem proves amb el General Log per veure si realment guarda les sentències.

    ```
    mysql> CREATE DATABASE provaLogs;
    Query OK, 1 row affected (0,01 sec)

    mysql> USE provaLogs;
    Database changed

    mysql> CREATE TABLE taulaProves (pepe INT);
    Query OK, 0 rows affected (0,02 sec)    
    ```

    I amb la comanda cat veurem el contingut del log general, on en teoria ens hauria de gravar les sentències anteriors.

    ```
    # cat /var/lib/mysql/general.log
    ```

    <details open>
    <summary><b>Mirar el contingut de General Log</b></summary>
    <img src="captures/generallogtest.png">
    </details>

4. Ara farem proves però amb el Slow Query Log, per veure si realment guarda les sentències lentes, utilitzarem la funció sleep per a que la sentència trigui més del temps mínim i registri la sentència com a lenta (La funció sleep triga 4 segons per cada registra que troba a la select).

    ```
    mysql> SELECT sleep(4), user FROM mysql.user;
    +----------+------------------+
    | sleep(4) | user             |
    +----------+------------------+
    |        0 | asix             |
    |        0 | grigor           |
    |        0 | mysql.infoschema |
    |        0 | mysql.session    |
    |        0 | mysql.sys        |
    |        0 | root             |
    +----------+------------------+
    6 rows in set (24,01 sec)
    ```

    I verificarem si ha enregistrat la consulta fent un cat al fitxer del Slow Query Log.

    ```
    # cat /var/lib/mysql/slow-query.log
    ```
    <details open>
    <summary><b>Mirar Contingut Slow Query</b></summary>
    <img src="captures/slowquerylogtest.png">
    </details>

## Desactivar els Logs binary, slow query i general
Ara per desactivar aquests logs (no borrar-los) sinó desactivar i que no es gravin més canvis ni en el fitxer de log Binary, Slow Query ni general.

1. Accedirem al fitxer de configuració de logs creat i configurat anteriorment `/etc/percona/logs.cnf`. 
    ```
    # nano /etc/percona/logs.cnf
    ```

2. Localitzarem l'apartat de General Log i el posarem en OFF/0.
    ```
    general_log = 0
    ```
    <details open>
    <summary><b>Desactivar el Log General</b></summary>
    <img src="captures/offgenerallog.png">
    </details>

3. Localitzarem l'apartat de Slow Query i el posarem en OFF/0
    ```
    slow_query_log = 0
    ```
    <details open>
    <summary><b>Desactivar el Log de Consultes Lentes</b></summary>
    <img src="captures/offslowquerylog.png">
    </details>

4. Desactivarem el log de Binaris. Com no ho hem configurat per activar-ho perquè ja venia per defecte activat, haurem d'afegir els següents paràmetres per desactivar-ho.
    ```
    disable_log_bin
    ```
    <details open>
    <summary><b>Desactivar el Log Binary</b></summary>
    <img src="captures/disablelogbin.png">
    </details>

5. Reiniciarem el Servei MySQLD comprovarem que el servei s'ha aixecat i comprovarem que els canvis s'han realitzat correctament.
    ```
    # systemctl restart mysqld

    # systemctl status mysqld
    ```
    <details open>
    <summary><b>Reiniciar Servei</b></summary>
    <img src="captures/reiniciar.png">
    </details>

    ```
    mysql> SHOW VARIABLES LIKE '%log';

    mysql> SHOW VARIABLES LIKE '%bin_log%';
    ```
    <details open>
    <summary><b>Comprovar que s'han desactivat els logs</b></summary>
    <img src="captures/comprovaroff.png">
    </details>

## Activar els Logs a Nivell de Sessió

Ara activarem els logs, però a nivell de sessió, a nivell de sessió vol dir que quan es reinici el servidor tornarà a l'estat anterior (com està configurat en el fitxer de configuracions de logs)

1. El log de Binaris, funciona diferent a la resta. Té una configuració **Master-Slave**. Hi ha un paràmetre pare (Master) que si no està activat, el paràmetre Slave no té utilitat, i aquest log és el únic log que **no es pot activar el Master a nivell de sessió**, només es pot activar a nivell de fitxer. 
Però si tenim activat el Master a nivell de fitxer, sí que podem activar i desactivar el slave a nivell de sessió i tendria sentit.

- Per activar el paràmetre Slave per tenir el Log de Binaris a nivell de sessió es faria de la següent manera:

    Nota: Hem de tenir el Master activat si volem canviar el Slave a nivell de sessió (si no, no té sentit).

    Paràmetre Màster = `log_bin`

    Paràmetre Slave = `sql_log_bin`

    ```
    mysql> SET sql_log_bin = ON;
    ```
    <details open>
    <summary><b>Activar el log de binaris (Slave) a nivell de sessió, (No podem activar el log_bin (Master) a nivell de sessió. Llavors en aquest cas no té cap utilitat.</b></summary>
    <img src="captures/sqllogbin.png">
    </details>

2. Ara, activarem el Slow Query Log a nivell de sessió:

- Alguns logs s'activen amb SET GLOBAL perquè són variables globals de sessió (per tots els usuaris) i sense globals només per la sessió corrents.

    ```
    mysql> SET GLOBAL slow_query_log = 'ON';
    ```
    <details open>
    <summary><b>Activar els Slow Query Log a nivell de sessió</b></summary>
    <img src="captures/sesionslow.png">
    </details>

3. Ara activarem el General Log amb el tipus de guardat de taula a nivell de sessió.

    ```
    mysql> SET GLOBAL log_output = "TABLE";
    ```
    <details open>
    <summary><b>Activar el guardar de logs tipus taula</b></summary>
    <img src="captures/outputtable.png">
    </details>

4. Activarem els logs generals a nivell de sessió

    ```
    mysql> SET GLOBAL general_log = ON;
    ```

    <details open>
    <summary><b>Activar el general log a nivell de taula</b></summary>
    <img src="captures/sesiongenerallog.png">
    </details>

- I farem qualsevol consulta perquè s'enregistri.

    <details open>
    <summary><b>Fer proves d'enregistrament</b></summary>
    <img src="captures/tests.png">
    </details>

5. Si ara consultem a la taula on es guarden els logs en format taula, podrem veure que es van registrant querys.

    Comprovarem que es guardin (però d'aquesta manera estan en format hexadecimal)
    ```
    mysql> SELECT * FROM mysql.general_log;
    ```

    <details open>
    <summary><b>Taula General_Log (Hexadecimal Arguments)</b></summary>
    <img src="captures/selectgenerallogtable.png">
    </details>

    Si volem veure les Selects en format String i no en format hexadecimal amb la següent select ho podrem veure, posant un \G ens ho mostrarà més ordenat.

    ```
    mysql> select a.*, convert(a.argument using utf8) from mysql.general_log a \G;
    ```

    <details open>
    <summary><b>Taula General_Log (En format Text Arguments)</b></summary>
    <img src="captures/texttable.png">
    </details>





 

