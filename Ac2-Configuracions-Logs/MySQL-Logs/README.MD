# Configuracions i comprovacions de logs en el MySQL <img align="right" width="70" src="../imatges/logs.ico"/>

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

1. A MySQL treballarem els diferents 4 tipus de logs:

    > - Logs d'Errors.
    > - Log General de Consultes.
    > - Log de Consultes Lentes.
    > - Log de Binaris.

    

    