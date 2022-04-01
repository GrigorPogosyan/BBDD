# Configuració Percona Server per fer Connexions Remotes Segures (SSL) <img align="right" width="70" src="../imatges/ssl_logo.png"/>

## Connexió per SSH al Sistema
Primer de tot, ens connectarem a la màquina per **SSH** desde CMD (ho tenim prèviament instal·lat).
```
ssh machineuser@ip 
```
<details open>
<summary><b>Connexió per SSH</b></summary>
<img src="../MySQL-Logs/captures/ssh.png">
</details>

<hr>

## [Extra] Accedir en Remot
Arribat al punt anterior, ja estaria la instal·lació i configuració del Percona Server, però aquest punt és per poder accedir-hi remotament.

Tal com ho tenim ara configurat:
- Port MySQL 3306 **escoltant**. ✔️
- Firewall Configurat. ❌
- Usuari MySQL amb el qual poder accedir-hi des de qualsevol màquina. ❌

1. Procedirem a configurar el Firewall per poder accedir al MySQL del Percona Server Remotament, configurarem la regla i reiniciarem el servei per a que s'apliquin els canvis.

```
# sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent

# sudo systemctl restart firewalld.service
```
<details open>
<summary><b>Configurar Firewall</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/configure_firewall.png">
</details>
<details open>
<summary><b>Veure configuració Firewall</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/check_firewall.png">
</details>

2. Ara procedirem a crear un usuari que pugui accedir des de qualsevol màquina al MySQL i li donarem els màxims privilegis possibles (per comprobar que funciona.)
```
# mysql -u root -p
[root password]

mysql> CREATE USER 'new_user'@'%' IDENTIFIED BY 'P4SSW0RD';

mysql> GRANT ALL PRIVILEGES ON *.* TO 'new_user'@'%';

mysql> FLUSH PRIVILEGES;
```
<details open>
<summary><b>Accés al MySQL</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/acced_to_mysql.png">
</details>

<details open>
<summary><b>Crear Usuari MySQL</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/create_user.png">
</details>

<details open>
<summary><b>Permisos Usuari MySQL</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/user_privileges.png">
</details>

<details open>
<summary><b>Flush Privileges perquè els canvis s'apliquin ara.</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/flush_privileges.png">
</details>

3. Crearem una connexió cap a aquesta màquina des del Workbench
<details open>
<summary><b>Creant connexió</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/connection_wb.png">
</details>
<details open>
<summary><b>Accedint al MySQL</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/connection_wb2.png">
</details>
<details open>
<summary><b>Testejant el SGBD</b></summary>
<img src="../../Ac1-Instal·lacions-SGBD/Percona-Server8.0/captures/create_db_test.png">
</details>
<hr>

## Comprovar amb el programa WireShark que la connexió d'autentificació al SGBD no és segura    <img src="captures/wire.png" width = 180 align=right>


Abans de les versions de MySQL 8.0, per defecte les connexions estaven desencriptats, i s'havien de configurar a mà per encriptar-los.

Ara, la versió que tenim de MySQL (8.0), posteriors a aquesta està per defecte activat amb encriptació TLS per raons de seguretat. A més a més, SSL es quedarà en desús, a partir d'ara quan veiem informació parlant de SSL es refereix a TLS.

Llavors el que farem, serà amb el client Workbench realitzar una connexió insegura, que ens donen una opció per fer això i amb el Wireshark capturarem les diferents sentències realitzades contra la BD.

1. El primer pas que haurem de fer serà descarregar el Wireshark. [Fent clic aquí podràs descarregar el Wireshark](https://www.wireshark.org/download.html)

    <details open>  
    <summary><b>Descarregar el WireShark per Windows x64</b></summary>
    <img src="captures/installwire.png">
    </details>

2. Realitzarem una connexió remota amb el Workbench amb la opció de no SSL no encriptat, per veure que sense la encriptació es poden veure les selects. 
    <details open>  
    <summary><b>Crear Connexió no SSL</b></summary>
    <img src="captures/nosslconnection.png">
    </details>

3. Obrirem el Wireshark i començarem a esnifar la xarxa. Com estem virtualitzant, haurem de seleccionar el adaptador virtual de xarxa que ens fa connectar amb la màquina virtual. En el meu cas la NAT VMNET19.
    <details open>  
    <summary><b>Seleccionar l'adaptador Wifi</b></summary>
    <img src="captures/vmnet19.png">
    </details>

4. Un cop dins veurem tots els missatges de la xarxa de l'adaptador que s'envien que es reben dels diferents dispositius, totes les peticions que es fan, amb les respotes...
    <details open>  
    <summary><b>Visió General Wireshark</b></summary>
    <img src="captures/generalvision.png">
    </details>

5. Ara, no volem veure tots els paquets, perquè en una xarxa gran hauriem d'estar hores buscant el paquet corresponent i resposta de la comunicació amb el MySQL llavors el Wireshark permet molts filtres de paquets, un d'ells és el d'especificar el port que utilitzarem. En aquest cas amb el filtre `tcp.port eq 3306` ens filtrarà per les comunicacions del MySQL. (En entorns de proves no ens apareixerà res perquè encara no hem fet la consulta, ve en el següent punt.)
    <details open>  
    <summary><b>Filtrar per comunicacions MySQL</b></summary>
    <img src="captures/portsql.png">
    </details>

6. Ara amb el client farem una consulta contra la BD de Sakila un d'exemple.
    <details open>  
    <summary><b>Consulta de Prova</b></summary>
    <img src="captures/testselect.png">
    </details>

7. Ara en el Wireshark buscarem els `REQUEST QUERY` i `RESPONSE QUERY`, el que l'usuari demana, i el que respon. On tenim el port filtrat hauria de ser més fàcil trobar-ho. Teòricament el Response estarà en la següent trama després de fer la consulta.
    <details open>  
    <summary><b>Request QUERY (Consulta que Fem)</b></summary>
    <img src="captures/requestquery.png">
    </details>

    <details open>  
    <summary><b>Response TABULAR (El que BD Retorna)</b></summary>
    <img src="captures/responsetabular.png">
    </details>

    I com s'ha pogut veure podem veure el contingut i això no és res segur ja que si fossin consultes i comunicacions de bancs ens poden robar diners.
<hr>

## Connexió per SSL (TLS) amb Certificats


