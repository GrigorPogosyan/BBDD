# Instal·lació Redis 5.0 <img align="right" width="165" src="../imatges/redis_logo.png"/>

## Connexió per SSH al Sistema
Primer de tot, ens connectarem a la màquina per **SSH** desde CMD (ho tenim prèviament instal·lat).
```
ssh machineuser@ip 
```
<details open>
<summary><b>Connexió per SSH</b></summary>
<img src="captures/ssh.png">
</details>

## Actualització dels Paquets del Sistema
Després, actualitzarem els nostres paquets del nostre sistema acceptant tots els inputs que ens vagi demanant.
```
# sudo yum update && yum upgrade
```
<details open>
<summary><b>Actualització Paquets SO</b></summary>
<img src="captures/updateupgrade.png">
</details>
<hr>

## Instal·lació Redis 5.0 des del Repositori
**Nota**: ***Totes les següents comandas s'han executat amb un usuari root***.

