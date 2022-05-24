# Configuració Sistemes de Rèplica<img align="right" width="185" src="../imatges/replica_logo.png"/>
## Connexió per SSH al Sistema (Opcional)
Ara, si al començar aquesta guía tenim configurada alguna IP en les nostres màquines i instal·lat el SSH. Accedirem en remot per una correcta administració del Sistema.
```
ssh usuarimaquina@ip
```
<details open>
<summary><b>Connexió per SSH al Master</b></summary>
<img src="captures/ssh.png">
</details>

<details open>
<summary><b>Connexió per SSH al Slave</b></summary>
<img src="captures/ssh2.png">
</details>


## Introducció
En aquesta guía aprendrem a crear un sistema de rèpliques amb Percona MySQL.
La tipologia que utilitzarem que és **Master Slave** en un entorn real podem d'utilitzar `1 Master` i `1 o més slaves` però per aquesta pràctica s'utilitzarà `1 Master` i `1 Slave`.

## Rèplica Master i Slave
El primer pas que hauriem de fer és configurar la xarxa dels servidors, per a que es vegin entre ells.
En el entorn virtual el propi VMWARE en mode NAT per DHCP ens proporciona connexió a Internet i visibilitat entre les màquines llavors ja ens estalviem aquest pas.

En un entorn real fent [click aqui](https://www.tecmint.com/set-static-ip-address-in-rhel-8/) podem veure com configurar la xarxa en els servidor Redhat 8.0 (RHEL85)
    

El següent pas que s'hauria de fer és tenir preparat les màquines amb Percona Server 8.0 amb el MySQL instal·lat. [Fent clic aqui](https://github.com/GrigorPogosyan/M02-Base-de-Dades/tree/main/Ac1-Instal%C2%B7lacions-SGBD/Percona-Server8.0) podràs veure la guia d'instal·lació de Percona-Server MySQL, necessitarem connexió a Internet en les màquines.

Ara, desactivarem el `SELINUX` en les `dues màquines` per estalviar problemes, en un cas real s'haurien de fer servir les regles. SELINUX és el Firewall de processos que té Red Hat per mesures de seguretat.

Anirem a l'arxiu del SELINUX i posarem en `disable` el valor.
```
# nano /etc/selinux/config
```
<details open>
<summary><b>Desactivarem SELINUX</b></summary>
<img src="captures/disable_selinux.png">
</details>

I posteriorment reiniciarem la màquina per aplicar els canvis.
```
# init 6
```

Ara, `desactivarem el Firewall` en les `dues màquines` per evitar més problemes, en un cas real s'haurien de configurar correctment les regles per seguretat.
```
# systemctl disable firewalld
```
<details open>
<summary><b>Desactivar el Firewall</b></summary>
<img src="captures/disable_firewalld.png">
</details>

Ara, en cas de que estiguem virtualitzant i dupliquem màquines de Percona, aquest té un UUID del server que serà el mateix i no hauria de ser-ho perquè ens donarà problemes. Si volem comprovar si tenen el mateix UUID o no, haurem de mirar-ho en el fitxer `auto.cnf` del MySQL en els dos servidors i comparar-los
```
# cat /var/lib/mysql/auto.cnf
```
<details open>
<summary><b>Comparar els UUID dels servidors</b></summary>
<img src="captures/compare_uuid.png">
</details>

En cas de que tinguem UUID iguals haurem de borrar el fitxer `auto.cnf` del MySQL del `Slave` reiniciar el servei i que torni a generar un nou `UUID`.
