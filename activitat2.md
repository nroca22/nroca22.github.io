# Cas pràctic 2

## Índex
* Instalar Apache
* Instalem MariaDB
* Creem la base de dades per Owncloud
* Instalem PHP i tots els moduls necessaris
* Instalem Owncloud


### Instalar Apache
Instalem el servidor Apache amb la seguent comanda:
`sudo apt install apache2`

![captura1](caspr2cap1.png)

Desactivem el llista de directoris del servidor:
`sudo sed -i "s/Options Indexes FollowSymLinks/Options FollowSymLinks/" /etc/apache2/apache2.conf`

![captura2](caspr2cap2.png)

### Instalem MariaDB
Instalem el gestor de bases de dades MariaDB:
`sudo apt-get install mariadb-server mariadb-client -y`

![captura3](caspr2cap3.png)

I configurem la nostra instalació

`sudo mysql_secure_installation`

![captura4](caspr2cap4.png)

Tot seguit, ens demanarà unes opcións de configuració;
* Change the root password: no
* Remove anonymous users?: y
* Disallow root login remotely?: y
* Remove test databases and acces to it?: y
* Reload privilege tables now?: y

Totes aquestes són opcións de seguretat.

Per ultim reiniciem el servei de MariaDB
`sudo systemctl restart mariadb.service`

![captura5](caspr2cap5.png)

### Creem la base de dades per Owncloud
Entrem a MariaDB

`sudo mysql -u root -p`

![captura6](caspr2cap6.png)

Creem la base de dades

`CREATE DATABASE owncloud;`

![captura7](caspr2cap7.png)

Creem l'usuari "owncloud" amb la seva contrasenya "Admin1234".

`CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'Admin1234';`

![captura8](caspr2cap8.png)

Li donem accés a l'usuari que acabem de crear a la base de dades de owncloud

`GRANT ALL ON owncloud.* TO 'ownclouduser'@'localhost' IDENTIFIED BY 'Admin1234' WITH GRANT OPTION;`

![captura9](caspr2cap9.png)

Apliquem els cambis amb  `FLUSH PRIVILEGES;` i `EXIT;` i sortim.

![captura10](caspr2cap10.png)

### Instalem PHP i tots els moduls necessaris
`sudo apt-get install software-properties-common -y`

![captura11](caspr2cap11.png)

`sudo add-apt-repository ppa:ondrej/php`

![captura12](caspr2cap12.png)

Actualitzem els paquets que acabem d'afegir amb
`sudo apt update`

![captura13](caspr2cap13.png)

Instalem PHP amb tots els moduls que necessitarem
`sudo apt install php7.4 libapache2-mod-php7.4 php7.4-common php7.4-mbstring php7.4-xmlrpc php7.4-soap php7.4-apcu php7.4-smbclient php7.4-ldap php7.4-redis php7.4-gd php7.4-xml php7.4-intl php7.4-json php7.4-imagick php7.4-mysql php7.4-cli php7.4-mcrypt php7.4-ldap php7.4-zip php7.4-curl -y`

![captura14](caspr2cap14.png)

Després de la instalació editem el fitxer php.ini i cambiem alguns dels seus valors
`sudo nano /etc/php/7.4/apache2/php.ini`

![captura15](caspr2cap15.png)

Els valors a cambiar són els seguents:
`file_uploads = On`
`allow_url_fopen = On`
`memory_limit = 256M`
`upload_max_filesize = 100M`
`display_errors = Off`
`date.timezone = Europe/Madrid`

### Instalem Owncloud
Descarregem la ultima versió del programa i descomprimim els fitxers, amés movem aquestos fitxers d'Owncloud a "/var/www/html/owncloud"

`cd /tmp && wget https://download.owncloud.com/server/stable/owncloud-complete-latest.zip`

![captura16](caspr2cap16.png)

`unzip owncloud-10.0.8.zip`

![captura17](caspr2cap17.png)

`sudo mv owncloud /var/www/html/owncloud/`

![captura18](caspr2cap18.png)

Cambiem el propietari i permís dels directoris d'Owncloud. www-data per a que els pugui utilitzar Apache, 755 para que els puga executar i llegir cualsevol usuari de linux.

`sudo chown -R www-data:www-data /var/www/html/owncloud/`

`sudo chmod -R 755 /var/www/html/owncloud/`

### Configurar /etc/hosts

![captura19](caspr2cap19.png)

Editarem el fiter hosts, que ens servirà per redirigir la adreça "owncloud.nrm,com" a 127.0.0.1, aquést fitxer actua de forma similar a un DNS:

`sudo nano /etc/hosts`

On afegirem la següent línia:

127.0.0.1     owncloud.NRM.com

![captura20](caspr2cap20.PNG)

Ara crearem l'arxiu owncloud.conf a la carpeta d'apache de sites-available:

`sudo nano /etc/apache2/sites-available/owncloud.conf`

I introduïrem les següents línies:

```
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html/owncloud/
    ServerName owncloud.nrm.com
    ServerAlias www.owncloud.nrm.com
    
   Alias /owncloud "/var/www/html/owncloud/"
    
   <Directory /var/www/html/owncloud/>
      Options +FollowSymlinks
      AllowOverride All
      Require all granted
        <IfModule mod_dav.c>
          Dav off
        </IfModule>
      SetEnv HOME /var/www/html/owncloud
      SetEnv HTTP_HOME /var/www/html/owncloud
    </Directory>
    
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

![captura21](caspr2cap21.PNG)

Després entrarem a la carpeta de sites-available:

![captura22](caspr2cap22.PNG)

I utilitzarem la comanda a2ensite amb el fitxer que acabem de crear, amb el paràmentre de sudo:

![captura23](caspr2cap23.PNG)

Tot següit reiniciarem el servei de apache2, amb `sudo service restart apache2` o amb `sudo systemctl restart apache2`

![captura24](caspr2cap24.PNG)

Ara comprovarém que apache s'hagi engegat correctament:

![captura25](caspr2cap25.PNG)

Si s'ha engegat correctament podem comprovar que ens redirigeixi correctament, introduïnt l'adreça al navegador:

![captura26](caspr2cap26.PNG)

### Configurar usuari i base de dades en owncloud

Necessitem crear un usuari i contrasenya d'administrador, indicar el directori de la base de dades que hem creat anteriorment, l'usuari, contrasenya, el nom, i el host:

usuari nroca  contrasenya nroca

![captura27](caspr2cap27.png)

Si tot està configurat correctament, auriem de poder veure la pàgina del gestor d'arxius d'owncloud

![captura28](caspr2cap28.png)

