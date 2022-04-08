# STORAGE ENGINES DEL MYSQL <img align="right" width="100" src="../imatges/storage_engine_logo.jpg"/>

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

## Motors d'Emmagatzematge Utilitzables
Per veure la informació de l'estat dels Storage Engines, és a dir per veure els motors suportats o no per defecte i veure quin és el que ve per defecte al crear noves taules, ho podrem fer de la següent manera des del MySQL.

El valor support indica si un motor d'emmagatzematge en particular està suportat o no i quin és el motor per defecte.

En aquest cas els que el support és NO, no podrem per defecte utilitzar aquest storage engine, haurem de fer un procés extra per activar-los.

Els que són support YES sí que els podrem utilitzar per defecte (actius).

I en el que posa DEFAULT és el que ve per defecte.

```
mysql> SHOW STORAGE ENGINES\G
```
<details open>
<summary><b>Veure Storage Engines</b></summary>
<img src="captures/show_storage_engines.png">
</details>
<details open>
<summary><b>Veure Storage Engines 2</b></summary>
<img src="captures/show_storage_engines2.png">
</details>

## Canviar el Motor d'Emmagatzematge per Defecte
Per veure ràpidament la informació dels Storage Engines, ho podem fer de la següent manera:

On podem veure que el Storage Engine per Defecte és el `InnoDB`.
```
mysql> SHOW ENGINES;
```
<details open>
<summary><b>Storage Engine per Defecte</b></summary>
<img src="captures/default_storage_engine.png">
</details>

Si vulgúessim canviar el Storage Engine per defecte, és a dir, que a partir d'ara totes les taules que es crein a la meva BBDD siguin per defecte amb un altre motor d'emmagatzematge, ho podrem fer de la següent manera.

Accedirem al fitxer de configuració de MySQL que es troba a la ruta `/etc/my.cnf`
```
# nano /etc/my.cnf
```

I afegiriem el següent paràmetre de configuració en les següents línies del `[MYSQLD]` indicant que estem configurant el dimoni del MySQL indicant quin és el Storage Engine per defecte (si no posem res per defecte és InnoDB). En aquest cas volem que per defecte utilitzi `MyISAM`
```
default-storage-engine=MyISAM
```
<details open>
<summary><b>Canviar el Storage Engine per defecte a MyISAM</b></summary>
<img src="captures/default_MyISAM.png">
</details>

I reiniciariem el servei de MySQL
```
# systemctl restart mysqld
```
<details open>
<summary><b>Reiniciar i Comprovar l'estat del MySQL</b></summary>
<img src="captures/reload_status_mysqld.png">
</details>

I ara si tornem a comprovar el Storage Engine per defecte, hauria de canviar el DEFAULT del InnoDB i s'ho posaria al Engine que hem configurat, en aquest cas el MyISAM.
```
mysql> SHOW ENGINES;
```
<details open>
<summary><b>Comprovació del Storage Engine per defecte MyISAM</b></summary>
<img src="captures/MyISAM_default.png">
</details>