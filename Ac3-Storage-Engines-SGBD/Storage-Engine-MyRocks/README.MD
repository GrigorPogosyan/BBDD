# STORAGE ENGINE MyRocks <img align="right" width="230" src="../imatges/myrocks_logo.png"/>

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

## Ús del Storage Engine MyRocks
### Introducció a MyRocks
MyRocks és un motor d'emmagatzematge basat en RocksDB. És de tipus Clau-Valor.

L'utilitza Facebook i han fet un estudi de rendiment amb InnoDB i han trobat diferents ventatges.

Les Advantatges a sobre del InnoDB són:

- <b><span style="color:green">Eficiència de Compressió:</span></b> Comprimeix millor perquè no utilitza mides de paginació fixes. Si en MyRocks una pàgina es comprimeix en 5 KB només utilitzarà aquest 5 KB d'emmagatzematge. Estalvia espai en disc i indirectament estalvia diners.

- <b><span style="color:green">Annexació i Compactació:</span></b> Utilitza menys quantitat de lectures i escriptures perquè els patrons d'accés a les dades estan emmagatzemats de manera persistents i són de clau-valor.

- <b><span style="color:green">Replicació més ràpida:</span></b> MyRocks no necessita lectures aleatories per actualitzar les claus primàries i no necessita verificar la integritat (replicació sense lectura)

- <b><span style="color:green">Carrega de dades més ràpid:</span></b> Activat aquesta opció a nivell de sessió MyRocks escriu dades directament en el nivell més baix, i evita tots els gastos de compactació que aquests utilitzen CPU, E/S per comprimir i descomprimir dades.

    [Més informació](https://www.codetd.com/es/article/12032793)

    [Més informació+](https://engineering.fb.com/2016/08/31/core-data/myrocks-a-space-and-write-optimized-mysql-database/)

    RocksDB s'utilitza àmpliament per emmagatzemar dades persistents en SSD a Facebook, i també l'utilitzen diversos serveis que serveixen consultes en línia als discs durs.


### Creació de BD amb Taules de Storage Engine MyRocks
Ara, crearem una BD on posteriorment crearem Taules fent servir el Storage Engine MyRocks.

```
mysql> CREATE DATABASE myRocks;
USE myRocks;
```
<details open>
<summary><b>Crear BD per myRocks</b></summary>
<img src="captures/create_myRocks.png">
</details>