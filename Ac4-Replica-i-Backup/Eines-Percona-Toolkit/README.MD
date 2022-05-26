# Ús de Percona Toolkit <img align="right" width="185" src="../imatges/percona_toolkit_logo.png"/>







PERCONA TOOLKIT

SI EN RHEL TENEMOS PORBLEMA DE REPOSITORIO, HACER LO SIGUIENTE:

dnf clean all
rm -frv /var/cache/dnf
subscription-manager refresh
dnf update

INSTALAMOS PERCONA TOOLKIT
sudo yum install percona-toolkit

Hacemos pt checksum, QUE LO QUE HACE ÉS MIRAR LA INTEGRIDAD DE LOS DATOS ENTRE MASTER Y SLAVE Y NOS DICE SI HAY ALGUNA DIFERENCIA ENTRE LA TABLA DEL MASTER Y SERVER
miramos el documentar perldoc y miramos el manual k tenems en linux
--nocheck-binlog-format

-d github    (--databases)

-h localhost
-u root
-p Sox2020$

--slave-user
--slave-password

 pt-table-checksum -u root -pSox2020$ -h localhost --no-check-binlog-format --slave-user=replication --slave-password=Sox2020$ --databases=github

 Si no pones --slave user ni slave password da error diiend k no se puede conectar

 ahora importamos sakila y hacemos esto 
  pt-table-checksum -u root -pSox2020$ -h localhost --no-check-binlog-format --slave-user=replication --slave-password=Sox2020$ --databases=sakila

  En el slave hacemos un update de actor por ejemplo, y luego volvemos a poner la comanda y nos dice que hay una diferencia al hacer checsum
  pt-table-checksum -u root -pSox2020$ -h localhost --no-check-binlog-format --slave-user=replication --slave-password=Sox2020$ --databases=sakila

TS ERRORS  DIFFS     ROWS  DIFF_ROWS  CHUNKS SKIPPED    TIME TABLE
05-25T06:33:00      0      1      200          0       1       0   0.317 sakila.actor

CLARO AHORA COMO ARREGLAMOS LOS DIFFS? PUES TENEMS LA EINA PT TALBE SYNC QUE PODEMOS DECIR QUE EL EN E CASO QUE HAYA DIFS QUE EL SLAVE VUELVA A SINCRONIZAR PERÒ HACIENDO CASO AL MASTER Y ESA TABLA SE HABRA ARREGLADO DE LOS DIFFS.

Ahora con  table sync miraremos en el servidor de replicacion que cambios han hecho que no estan en el master
pt-table-sync --sync-to-master h=192.168.151.142,u=replication,p=Sox2020$,P=3306 --databases=sakila --tables=actor --print

una vez que miramos los cambios, hacemos la sync, no miramos check de cpnstraint ni tablas de hijos:

pt-table-sync --sync-to-master h=192.168.151.142,u=replication,p=Sox2020$,P=3306 --databases=sakila --tables=actor --no-check-child-tables --no-foreign-key-checks --execute



sino nos da este problema, aunque bueno, estando en el master no tenemos que tenre los datos mal y nos podemos fiar y hacer quen o compruebe las foraneas ya qeu en el master me lo habran introducido bien

pt-table-sync --sync-to-master h=192.168.151.142,u=replication,p=Sox2020$,P=3306 --databases=sakila --tables=actor --execute
Cannot delete or update a parent row: a foreign key constraint fails (`sakila`.`film_actor`, CONSTRAINT `fk_film_actor_actor` FOREIGN KEY (`actor_id`) REFERENCES `actor` (`actor_id`) ON DELETE RESTRICT ON UPDATE CASCADE) [for Statement "REPLACE INTO `sakila`.`actor`(`actor_id`, `first_name`, `last_name`, `last_update`) VALUES ('1', 'PENELOPE', 'GUINESS', '2006-02-15 04:34:33') /*percona-toolkit src_db:sakila src_tbl:actor src_dsn:P=3306,h=192.168.151.143,p=...,u=replication dst_db:sakila dst_tbl:actor dst_dsn:P=3306,h=192.168.151.142,p=...,u=replication lock:1 transaction:1 changing_src:1 replicate:0 bidirectional:0 pid:29192 user:root host:localhost.localdomain*/"] at line 10950 while doing sakila.actor on 192.168.151.142

I despres si tornem a fer checksum ja no tindrem diferencies

pt-table-checksum -u root -pSox2020$ -h localhost --no-check-binlog-format --slave-user=replication --slave-password=Sox2020$ --databases=sakila