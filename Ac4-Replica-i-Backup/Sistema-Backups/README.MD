# Backups amb Percona XtraBackup <img align="right" width="170" src="../imatges/percona_xtra_logo.png"/>

## Connexió per SSH al Sistema (Opcional)
Si tenim instal·lat SSH en la nostra màquina de `MySQL`. Accedirem en remot per una correcta administració del Sistema.
```
ssh usuarimaquina@ip
```
<details open>
<summary><b>Connexió per SSH a la Màquina</b></summary>
<img src="captures/ssh.png">
</details>

## Introducció
En aquest apartat, farem una còpia completa i una restauració de les nostres bases de dades i del directori de dades.

Tindrem un servidor MySQL, on farem un Backup dels fitxers que té, després farem canvis en les bases de dades que té, eliminació de base de dades, updates, simulant que hi ha hagut una incidència, i que volem tornar a recuperar les dades restaurant el backup anterior. A més 

Utilitzarem l'eina `Percona XtraBackup` que alguns dels següents scripts o programes són els següents:

 - `xtrabackup` --> L'eina que utilitzarem per fer la còpia i la restauració del Backup.

 - `xbcrypt` --> Eina que encripta i desencripta fitxers de Backup.

