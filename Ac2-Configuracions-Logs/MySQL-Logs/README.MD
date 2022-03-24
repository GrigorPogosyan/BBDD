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

1. Crearem la estructura de path

    