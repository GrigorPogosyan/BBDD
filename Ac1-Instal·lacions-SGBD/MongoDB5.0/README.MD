# Instal·lació Mongo DB 5.0 <img align="right" width="130" src="../imatges/mongo_logo.png"/>

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
<img src="captures/upgrade.png">
</details>
<hr>

## Instal·lació MongoDB 5.0 des del Repositori
**Nota**: ***Totes les següents comandas s'han executat amb un usuari root***.

1. Començarem afegint el Repositori de Mongo DB 5.0 al nostre Linux.
    
    Crearem el repositori de Mongo DB 5.0

    ```
    # sudo nano /etc/yum.repos.d/mongodb-org-5.0.repo
    ```

    I dins afegirem lo següent per a que es connecti als repositoris de Mongo DB 5.0

    ```
    [mongodb-org-5.0]
    name=MongoDB Repository
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/5.0/x86_64/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-5.0.asc
    ```

    <details open>
    <summary><b>Configuració Repositori Mongo DB 5.0</b></summary>
    <img src="captures/repo_mongo.png">
    </details>
    

    Posteriorment actualitzarem els paquets perquè es refresquin els repositoris actuals i podrem veure que hi ha un de nou que és el que hem afegit.
    ```
    # yum update
    ```
    <details open>
    <summary><b>Actualitzar Paquets</b></summary>
    <img src="captures/update_repo.png">
    </details>

2. Ara un cop tenim el repositori configurat podrem instal·lar el MongoDB en la versió més estable en aquest cas 5.0

    ```
    # sudo yum install -y mongodb-org 
    ```
    <details open>
    <summary><b>Instal·lació MongoDB</b></summary>
    <img src="captures/install_mongo.png">
    </details>
    ...

    <details open>
    <summary><b>Instal·lació Feta MongoDB</b></summary>
    <img src="captures/install_mongo2.png">
    </details>
<hr>

## Post Instal·lació

1. Un cop tenim instal·lat el MongoDB, ens assegurarem que en els directoris de dades i de logs el propietari és el usuari mongod que és el que utilitza per defecte MongoDB per la instal·lació i creació de dependencies.
    ```
    # ls -ld /var/lib/mongo

    # ls -ld /var/log/mongodb
    ```
    <details open>
    <summary><b>Comprovació Permisos</b></summary>
    <img src="captures/owner.png">
    </details>

2. Ara iniciarem el Servei de MongoDB i comprovarem que hagi arrancat.

    ```
    # sudo systemctl start mongod

    # sudo systemctl status mongod
    ```

    <details open>
    <summary><b>Activar Servei mongod</b></summary>
    <img src="captures/start_status_mongod.png">
    </details>

3. Ara habilitarem el servei de MongoDB per a que s'inicii juntament amb el sistema operatiu.
    ```
    # sudo systemctl enable mongod
    ```

    <details open>
    <summary><b>Iniciar mongod amb el sistema</b></summary>
    <img src="captures/enable_mongod.png">
    </details>

4. Arribat en aquest punt, si volem accedir la shell de MongoDB des de local podrem utilitzar la comanda **mongosh**, on es la manera més ràpida de conectar-nos, configurar i treballar amb la nostra base de dades. Per defecte ens situara a la BD de Test que crea per defecte, a més veurem la versió de MongoDB instal·lat. Abans per accedir a la shell era posant **mongo** només, però ho estan pasant a mongosh perquè és mes visual i pràctic.

    ```
    # mongosh
    ```
    <details open>
    <summary><b>Accedir a la Shell de MongoDB + Versió</b></summary>
    <img src="captures/mongosh.png">
    </details>
<hr>

## [Extra] Accedir a MongoDB en Remot
Arribat al punt anterior tindriem el MongoDB instal·lat i "configurat" de base. 

Ara ens agradaria accedir i gestionar aquesta BD en remot. 

El client que utilitzarem serà **[Studio 3T](https://studio3t.com/knowledge-base/articles/installation/)** <img src="captures/studio3t.jpg" width=100 align="right"/>
<hr>

### Configuració Servidor
1. Lo primer que farem serà crear un usuari administrador i li assignarem per defecte la taula admin.

    ```
    # mongosh

    test> use admin
	      db.createUser(
  	        {
                user: "myServerAdmin",
                pwd: "password",
                roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  	        }
	); 
    ```
    <details open>
    <summary><b>Crear usuari Administrador</b></summary>
    <img src="captures/admin_user.png">
    </details>

2. Accedirem a l'arxiu de configuració de MongoDB, on canviarem el bind_ip per la IP del Servidor, també podem canviar el port per un altre per major seguretat però no el farem en aquest cas. Guardarem i sortirem.
    ```
    # nano /etc/mongod.conf
    bindIp = SERVERIP
    ```
    <details open>
    <summary><b>Canviar el BindIp</b></summary>
    <img src="captures/bindip.png">
    </details>

3. Ara afegirem la regla de firewall per obrir el port del MongoDB que tinguem configurat perquè accepti peticions entrant.

    ``` 
    # firewall-cmd --zone=public --add-port=27017/tcp --permanent
    ```

    <details open>
    <summary><b>Configuració Firewall</b></summary>
    <img src="captures/firewall.png">
    </details>

4. Reiniciarem el servei de MongoDB
    ```
    # sudo systemctl restart mongod
    ```

5. Si per alguna raó dona error al reiniciar-se, i en els logs surt el següent error:
    ```
    Failed to unlink socket file","attr":{"path":"/tmp/mongodb-27017.sock","error":"Oper..
    ```
    Haurem d'eliminar el següent fitxer:
    ```
    sudo rm /tmp/mongodb-27017.sock
    ```
    Aquest error es degut a que estem reiniciant el servei de mongoDB amb un usuari que no té accés de lectura al PATH `/var/lib/mongo` i al eliminar el fitxer es torna a crear però amb nous permisos
    
    [Més informació+](https://www.dev2qa.com/how-to-fix-failed-to-unlink-socket-file-error-when-start-mongo-db-on-macos/)

6. Ara si volem accedir a la Base de Dades en local ho farem de la següent forma depenent del usuari:
    ```
    mongo --host x.x.x.x --port xxxxx -u user -p password --authenticationDatabase YourAssignedDB 
    ```
    <details open>
    <summary><b>Conneció Mongosh</b></summary>
    <img src="captures/mongosh_access.png">
    </details>
7. Un cop arribat en aquest moment, el nostre URI String per accedir en remot des de l'aplicació és aquest URI el que fa és omplir els camps de configuració automàticament, però nosaltres ho farem a mà en els següents passos:

    `mongodb://myUser:12345678@x.x.x.x:xxxxx/myDatabase`

### Configuració Client
Ara procedirem a configurar el client, accedir a la BD i fer proves per veure que funciona.

1. Crearem una nova connexió en el programa client esmentat anteriorment.

    <details open>
    <summary><b>Nova Connexió</b></summary>
    <img src="captures/new_connection.png">
    </details>

2. Configurarem les següents finestres depenent de la configuració del nostre servidor.

    <details open>
    <summary><b>Pestanya Server: Tipus de connexió i la IP del nostre Servidor</b></summary>
    <img src="captures/server_connect.png">
    </details>

    <details open>
    <summary><b>Pestanya Autenticació: Mode d'Autenticació, nom, contrasenya i base de dades assignada a l'usuari.</b></summary>
    <img src="captures/auth_connection.png">
    </details>

    <details open>
    <summary><b>Pestanya SSH: Crearem un túnel per SSH i testejarem la connexió</b></summary>
    <img src="captures/ssh_connection.png">
    </details>

    <details open>
    <summary><b>Test Connection: I si tot va bé ens dirà que pot connectar-se</b></summary>
    <img src="captures/connection_succesful.png">
    </details>

3. Un cop tinguem la configuració correcta crearem la connexió i ens connectarem.
    <details open>
    <summary><b>Connexió</b></summary>
    <img src="captures/connection_manager.png">
    </details>

4. Obrirem una Shell i farem proves (crear base de dades, crear collections i insertar collections) per veure que funciona la connexió...
    <details open>
    <summary><b>Proves en el SGBD</b></summary>
    <img src="captures/testing_db.png">
    </details>

I ja tindriem la instal·lació i postinstal·lació feta.

<hr>

## WEBGRAFIA
