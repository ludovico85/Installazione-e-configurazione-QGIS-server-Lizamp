# Installazione-e-configurazione-Lizamp e QGIS Server

![GitHub last commit](https://img.shields.io/github/last-commit/ludovico85/Installazione-e-configurazione-QGIS-server-Lizamp?color=green&style=plastic)

Il repository contiene le istruzioni per l'installazione di QGIS server e Lizmap su server Ubuntu.

La documentazione ufficiale di Lizmap può essere consultata al [link](https://docs.lizmap.com/current/it/index.html)

Bisogna aver configurato il server e installato apache2 (postgresql opzionale) (si può consultare [il repo](https://github.com/ludovico85/Installazione-e-configurazione-di-postgresql-e-postgis-su-server-ubuntu-20.04/blob/master/README.md))

## Installazione di QGIS server

https://qgis.org/en/site/forusers/alldownloads.html#linux

Lizmap utilizza QGIS Server per la distribuzione dei dati sottoforma di servizi OGC. Prerequisito fondamentale è l'installazione di QGIS Server.

Per installare QGIS Server prima è necessario installare alcuni tool:

```
sudo apt install gnupg software-properties-common
```
Successivamente è necessario installare la chiave di installazione QGIS dal repository:

```
wget -qO - https://qgis.org/downloads/qgis-2021.gpg.key | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/qgis-archive.gpg --import
```
```
sudo chmod a+r /etc/apt/trusted.gpg.d/qgis-archive.gpg
```
Adesso è necessatrio aggiungere il repository e aggiornare i pacchetti di ubuntu:

```
sudo add-apt-repository "deb https://qgis.org/ubuntu-ltr $(lsb_release -c -s) main"
sudo apt update
```
Installare qgis-server e alcuni plugin:

```
sudo apt install qgis-server libapache2-mod-fcgid --no-install-recommends --no-install-suggests
sudo apt install python-qgis
```

---

**NOTA BENE**
Può capitare di ricevere un errore
```
dpkg: error processing package qgis-providers (--configure):
installed qgis-providers package post-installation script subprocess returned
error exit status 134
dpkg: dependency problems prevent configuration of python3-qgis:
python3-qgis depends on qgis-providers (= 1:3.16.3+32focal); however:
Package qgis-providers is not configured yet
```

Questo può succedere perché si sta tentando di installare una versione non compatibile con la distribuzione attuale di ubuntu.
https://askubuntu.com/questions/1309161/dpkg-error-processing-package-qgis-providers-configure

In questo caso:
1. Effettura la pulizia dei pacchetti

```
sudo apt remove *qgis*
sudo apt autoremove
sudo apt upgrade
```
2. Disabilitare i repository ```sudo nano /etc/apt/sources.list```
3. Utilizzare ubuntugis:
```
sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
sudo apt-get update
sudo apt install qgis-server libapache2-mod-fcgid --no-install-recommends --no-install-suggests
sudo apt install python3-qgis
```

---

Abilitare il modulo di apache fcgid e abilitare lo script serve-cgi-bin
```
sudo a2enmod fcgid
sudo a2enconf serve-cgi-bin
```
Riavviare apache2:
```
sudo service apache2 restart
```
Per testare l'installazione:
```
/usr/lib/cgi-bin/qgis_mapserv.fcgi
```
Creare il servizio di configurazione di QGIS server:
```
sudo touch /etc/apache2/sites-available/qgis-server.conf
sudo nano /etc/apache2/sites-available/qgis-server.conf
```
Incollare il seguente contenuto:
```
<VirtualHost *:80>
  ServerAdmin webmaster@localhost
  ServerName qgis-server

  DocumentRoot /var/www/html

  # Apache logs (different than QGIS Server log)
  ErrorLog ${APACHE_LOG_DIR}/qgis-server-error.log
  CustomLog ${APACHE_LOG_DIR}/qgis-server-access.log combined

  # Longer timeout for WPS... default = 40
  FcgidIOTimeout 120

  FcgidInitialEnv LC_ALL "en_US.UTF-8"
  FcgidInitialEnv PYTHONIOENCODING UTF-8
  FcgidInitialEnv LANG "en_US.UTF-8"

  # QGIS log
  FcgidInitialEnv QGIS_SERVER_LOG_STDERR 1
  FcgidInitialEnv QGIS_SERVER_LOG_LEVEL 0

  # default QGIS project
  SetEnv QGIS_PROJECT_FILE /home/qgis/projects/world.qgs

  # QGIS_AUTH_DB_DIR_PATH must lead to a directory writeable by the Server's FCGI process user
  FcgidInitialEnv QGIS_AUTH_DB_DIR_PATH "/home/qgis/qgisserverdb/"
  FcgidInitialEnv QGIS_AUTH_PASSWORD_FILE "/home/qgis/qgisserverdb/qgis-auth.db"

  # Set pg access via pg_service file
  SetEnv PGSERVICEFILE /home/qgis/.pg_service.conf
  FcgidInitialEnv PGPASSFILE "/home/qgis/.pgpass"

  # if qgis-server is installed from packages in debian based distros this is usually /usr/lib/cgi-bin/
  # run "locate qgis_mapserv.fcgi" if you don't know where qgis_mapserv.fcgi is
  ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
  <Directory "/usr/lib/cgi-bin/">
    AllowOverride None
    Options +ExecCGI -MultiViews -SymLinksIfOwnerMatch
    Order allow,deny
    Allow from all
    Require all granted
  </Directory>

 <IfModule mod_fcgid.c>
 FcgidMaxRequestLen 26214400
 FcgidConnectTimeout 60
 </IfModule>

</VirtualHost>
```
Creare alcune cartelle che ospiteranno i logs di QGIS Server, il database di autenticazione e i progetti:
```
sudo mkdir -p /var/log/qgis/
sudo chown www-data:www-data /var/log/qgis
sudo mkdir -p /home/qgis/qgisserverdb
sudo chown www-data:www-data /home/qgis/qgisserverdb
sudo mkdir -p /home/qgis/projects
sudo chown www-data:www-data /home/qgis/projects
```
Aggiungere lo script a2dissite per disabilitare il virtual host di default:
```
sudo a2dissite 000-default.conf
```
Ora abilitare il file di configurazione del virtual host di qgis-server:
```
sudo a2ensite qgis-server.conf
```
Verificare la correttezza della sintassi:

```
sudo apachectl configtest
```
Attivare il virtual host:
```
sudo systemctl restart apache2
```
Scarica il progetto demo:
```
cd /home/qgis/projects/
sudo wget https://github.com/qgis/QGIS-Training-Data/archive/release_3.16.zip
sudo unzip release_3.16.zip
sudo mv QGIS-Training-Data-release_3.16/exercise_data/qgis-server-tutorial-data/world.qgs .
sudo mv QGIS-Training-Data-release_3.16/exercise_data/qgis-server-tutorial-data/naturalearth.sqlite .
```
Testare la corretta installazione di QGIS server digitando nel browser:
```
http://myhost/cgi-bin/qgis_mapserv.fcgi?SERVICE=WMS&VERSION=1.3.0&REQUEST=GetCapabilities
```
La risposta dovrebbe essere simile all'immagine qui sotto.

![Alt text](/img/xml.png)
Per accedere a log di qgis-server
```
sudo nano /var/log/apache2/qgis-server-error.log
sudo nano /var/log/apache2/qgis-server-access.log
```
## Installazione di Lizmap

Le seguenti istruzioni si riferiscono alla versione 3.4 di Lizmap.

Aggiornare i pacchetti di ubuntu:

```
sudo apt update
```

Lizmap è basato sul framework PHP Jelix. Necessita l'installazione dei seguenti pacchetti (assicurasi che la versione di php sia quella corrente):


```
sudo apt install xauth htop curl libapache2-mod-fcgid libapache2-mod-php7.4 php7.4-cgi php7.4-gd php7.4-sqlite3 php7.4-curl php7.4-xmlrpc python-simplejson software-properties-common php7.4-xml
```

### Installazione del WebClient

Spostarsi nella cartella /var/www

```
cd /var/www
```
Creare la variabile per la versione da installare:

```
VERSION=3.4.7
```

Scaricare i file necessari:

```
sudo wget https://github.com/3liz/lizmap-web-client/releases/download/$VERSION/lizmap-web-client-$VERSION.zip

sudo unzip lizmap-web-client-$VERSION.zip

sudo ln -s /var/www/lizmap-web-client-$VERSION/lizmap/www/ /var/www/html/lizmap

sudo rm lizmap-web-client-$VERSION.zip
```

Copiare e rinominare i seguenti file:

```
cd /var/www/lizmap-web-client-$VERSION/lizmap/var/config
sudo cp lizmapConfig.ini.php.dist lizmapConfig.ini.php
sudo cp localconfig.ini.php.dist localconfig.ini.php
sudo cp profiles.ini.php.dist profiles.ini.php
cd ../../..
```

Per abilitare il repository DEMO aggiungere al file localconfig.ini.php le stringhe (dopo put here....):
```
[modules]
lizmap.installparam=demo
```

```
sudo nano localconfig.ini.php
```

Lanciare l'installazione:

```
sudo php lizmap/install/installer.php
```

Lanciare lo scritp set_rights:

```
cd /var/www/lizmap-web-client-$VERSION/
sudo lizmap/install/set_rights.sh www-data www-data
```

In caso non funzionasse, settare i permessi alla cartella di progetto di Lizmap

```
cd /var/www/lizmap-web-client-$VERSION/lizmap
sudo chown -R www-data:www-data qgis_projects
```

Riavviare apache2

```
sudo systemctl restart apache2
```

Per verificare il funzionamento digitare nel browser http://my_host/lizmap

Il firewall potrebbe bloccare le connessioni. Ablitare le connessioni sulla porta 80

```
sudo ufw allow 80/tcp
```

## Lizmap Extra
### Redirect
```
sudo ln -s /var/www/lizmap-web-client-$VERSION/lizmap/www/ /var/www/html/nome
```
### Geolocalizzazione
Se si vuole utilizzare la funzione di geolocalizzazione è necesario che lo scambio dei dati avvenga attraverso protoclli certificati e sicuri quali l'HTTPS. Bisogna quindi configurare apache2 affinchè possa utilizzare il protocollo. A questo [link](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04) viene riportata una possibile soluzione.
### Gestione dei dati tramite PostGIS
Se si vuole utilizzare Postgreslq/PostGIS per la gestione dei dati, basta installare il seguente modulo (dopo aver installato postgresql e l'estensione postgis):
```
sudo apt-get install php7.4-pgsql
```
### creare una nuova cartella per i repository
```
cd /var/www/lizmap-web-client-$VERSION/lizmap
sudo mkdir qgis_projects
```
Cambiare i permessi per l'utente
```
sudo chown user:group /var/www/lizmap-web-client-3.4.0/lizmap/qgis_projects
```
### Abilitare l'esportazione di altri formati di dati
https://github.com/3liz/qgis-wfsOutputExtension
1) Scaricare, decomprimere e copiare l'intera cartella `wfsOutputExtension` nella cartella d'installazione dei plugin di qgis-server `/usr/share/qgis/python/plugins`
```
cd /usr/share/qgis/python/plugins
sudo wget https://github.com/3liz/qgis-wfsOutputExtension/releases/download/1.6.2/wfsOutputExtension.1.6.2.zip
sudo unzip wfsOutputExtension.1.6.2.zip
sudo rm wfsOutputExtension.1.6.2.zip
```
2) settare i diritti (Da verificare)

2) Riavviare apache
```
sudo systemctl restart apache2
```
### Abilitazione di Spatialite
Per abilitare Spatialite è necessario:
1) Verificare che php sqlite sia correttamente installato
```
php7.4 -m|grep sqlite
```
```
pdo_sqlite
sqlite3
```
2) Installare il modulo spatialite
```
sudo apt update
sudo apt install libsqlite3-mod-spatialite
```
3) Modificare il file di configurazione php.ini.
```
sudo nano /etc/php/7.4/apache2/php.ini
```
4) Sostituire la configurazione di default:
```
[sqlite3]
;sqlite3.extension_dir =
```
con la seguente configurazione:
```
[sqlite3]
sqlite3.extension_dir = /var/www/sqlite3_ext
```
5) Creare una directory in /var/www/ sqlite3_ext
```
sudo mkdir /var/www/sqlite3_ext
```
6) Verificare le versioni delle librerie libspatialite.so.7.1.x e mod_spatialite.so.7.1.x
```
sudo ls /usr/lib/x86_64-linux-gnu/
```
7) Copiare e incollare la libreria libspatialite.so.7.1.x e mod_spatialite.so.7.1.x
```
sudo cp /usr/lib/x86_64-linux-gnu/libspatialite.so.7.1.1 /var/www/sqlite3_ext
sudo cp /usr/lib/x86_64-linux-gnu/mod_spatialite.so.7.1.0 /var/www/sqlite3_ext
sudo cp /usr/lib/x86_64-linux-gnu/mod_spatialite.so.7 /var/www/sqlite3_ext
sudo cp /usr/lib/x86_64-linux-gnu/mod_spatialite.so /var/www/sqlite3_ext
sudo cp /usr/lib/x86_64-linux-gnu/libspatialite.so.7 /var/www/sqlite3_ext
```
8) Riavviare apache
```
sudo systemctl restart apache2
```
## Configurare invio mail
Per configurare l'invio delle email è necessacio modificare il file localconfig.ini.php. Con gmail è necessario abilitare le app non sicure https://accounts.google.com/DisplayUnlockCaptcha
**Non utilizzare caratteri speciali nella password.**
```
sudo nano /var/www/lizmap-web-client-$VERSION/lizmap/var/config/localconfig.ini.php
```
```
[mailer]
webmasterEmail="XXX@gmail.com"
webmasterName="XXX"

; how to send mail : "mail" (mail()), "sendmail" (call sendmail), or "smtp" (send directly to a smtp)
mailerType=smtp
; Sets the hostname to use in Message-Id and Received headers
; and as default HELO string. If empty, the value returned
; by SERVER_NAME is used or 'localhost.localdomain'.
hostname=
sendmailPath="/usr/sbin/sendmail"

; if mailer = smtp , fill the following parameters

; SMTP hosts.  All hosts must be separated by a semicolon : "smtp1.example.com:25;smtp2.example.com"
smtpHost="smtp.gmail.com"
; default SMTP server port
smtpPort=587
; secured connection or not. possible values: "", "ssl", "tls"
smtpSecure="tls"
; SMTP HELO of the message (Default is hostname)
smtpHelo=
; SMTP authentication
smtpAuth=on
smtpUsername="XXX@gmail.com"
smtpPassword="mypassword"
; SMTP server timeout in seconds
smtpTimeout=30

```
## Temi personalizzati
Scaricare una copia del tema di default digitando nella barra di ricerca del browser l'indirizzo http://my_host/lizmap/index.php/view/media/getDefaultTheme . Decomprimere la cartella e copiare l'intero contenuto all'interno di uno dei repository (il tema modificato si applica all'intero repository), creando una cartella media/themes/
La struttura finale è la seguente:

    -- media
        |-- themes
            |-- default
            |-- map_project_file_name1
            |-- map_project_file_name2
            |-- etc

Di default applica il tema a tutti i progetti presenti nel repository.

## Template personalizzati
Lizmap è basato su [Jelix 1.6](https://docs.jelix.org/en/manual-1.6/templates#redefining-a-template). Il tema di default si trova nella cartella `/var/www/lizmap-web-client-3.4.7/lizmap/var/themes/default`. I template dei vari moduli si trovano nella cartella `/var/www/lizmap-web-client-3.4.7/lizmap/modules`. Per modificare un template bisogna creare una copia del template all'interno della cartella default.
Esempio se si vuole personalizzare il template main.tpl del modulo view, si crea la cartella view all'interno del tema default e si copia il template main.tpl

```
sudo mkdir /var/www/lizmap-web-client-3.4.7/lizmap/var/themes/default/view

sudo cp /var/www/lizmap-web-client-3.4.7/lizmap/modules/view/templates/main.tpl /var/www/lizmap-web-client-3.4.7/lizmap/var/themes/default/view
```
Adesso è possibile modificare il template di default

```
sudo nano /var/www/lizmap-web-client-3.4.7/lizmap/var/themes/default/view/main.tpl
```

## Installazione di un X server
Può capitare di ricevere un errore nella chaiamata della stampa (GetPrint). Per risolvere è necessario installare un X server e configurarlo correttamente.
```
sudo apt-get install xvfb
```
Creare e modificare il file xvfb.service:
```
sudo touch /etc/systemd/system/xvfb.service
sudo nano /etc/systemd/system/xvfb.service
```
Incollare il seguente contenuto:
```
[Unit]
Description=X Virtual Frame Buffer Service
After=network.target

[Service]
ExecStart=/usr/bin/Xvfb :99 -screen 0 1024x768x24 -ac +extension GLX +render -noreset

[Install]
WantedBy=multi-user.target
```
Abilitare e verificare lo stato del servizio:
```
sudo systemctl enable --now xvfb.service
sudo systemctl status xvfb.service
```

## Configurazione del file pg_service.conf
Struttura del file pg_service.conf

```
[nomeservizio]
host=XX.XXX.XXX.XX
port=XX
dbname=XXXXXXXX
user=XXXXXXXXXX
password=**********
```
### Windows
- Creare il file pg_service.conf in una cartella
- impostare la variabile dell'utente PGSERVICEFILE con il percorso al file pg_service.conf

### UBUNTU Server
- Creare il file pg_service.conf nella cartella ```sudo nano /etc/postgresql-common/pg_service.conf```
- Testare il funzionamento
```
psql postgresql://?service=nome_servizio
```
Per verificare la connessione in atto
```
nomedatabase=> \q
```
```
=> You are now connected to database "***********" as user "**************".
```
Ora bisogna configurare QGIS-SERVER e Apache2 affinchè vedano il file di servizio

```
sudo nano /etc/apache2/sites-available/qgis-server.conf
```
Modificare il file di configurazione in questo modo

```
# Set pg access via pg_service file
SetEnv PGSERVICEFILE /etc/postgresql-common/pg_service.conf
SetEnv PGSERVICE=nomeservizio
```
Riavviare Apache2
```
sudo systemctl restart apache2
```
Verificare che il file pg_service.conf abbia i permessi di lettura. In caso contratrio
```
sudo chmod 644 /etc/postgresql-common/pg_service.conf
```
In questo modo il proprietario può leggere e scrivere (rw-), il gruppo può leggere solo (r--), gli altri possono leggere solo (r--). Per mostrare i permessi
```
ls -l /etc/postgresql-common
```
Il che darà:
```
-rw-r--r-- 1 root root  108 Feb 16 09:46 pg_service.conf
```
## Upgrade dalla versione 3.5 alla versione 3.7
Creare una copia di sicurezza dell'installazione precedente
```
cd /var/www/
sudo cp -r lizmap-web-client lizmap-web-client.old
```

Lanciare lo script backup.sh
```
sudo lizmap/install/backup.sh /tmp
```
Scaricare la nuova versione
```
cd /var/www
VERSION=3.7.6
sudo wget https://github.com/3liz/lizmap-web-client/releases/download/$VERSION/lizmap-web-client-$VERSION.zip
sudo unzip lizmap-web-client-$VERSION.zip
```
Rinominare la vecchia cartella di lizamp
```
cd lizmap-web-client
sudo mv lizmap lizmap.bak
```
Copiare la nuova cartella lizmap nella vecchia direcotry e lanciare lo script restore
```
sudo cp -r ../lizmap-web-client-$VERSION/lizmap lizmap
sudo lizmap/install/restore.sh /tmp
```

Lanciare l'installazione della nuova versione
```
sudo lizmap/install/clean_vartmp.sh
sudo php lizmap/install/configurator.php
sudo php lizmap/install/installer.php
```
Plulizia della cache e di tutti i file temporanei
```
sudo lizmap/install/clean_vartmp.sh
```
Lanciare lo script per settare i permessi utente
```
sudo lizmap/install/set_rights.sh www-data www-data
```
Importare i vecchi progetti
```
cd /var/www/
sudo cp -r lizmap-web-client.old/lizmap-web-client/lizmap/qgis_projects lizmap-web-client/lizmap/qgis_projects
```
### Installare qgis-server plugin (obbligatorio)
Spostarsi nella cartella dei plugins e scaricare il file
```
cd /usr/share/qgis/python/plugins
sudo wget https://github.com/3liz/qgis-lizmap-server-plugin/archive/refs/tags/2.9.0.zip
sudo unzip 2.9.0.zip
sudo cp -r qgis-lizmap-server-plugin-2.9.0/lizmap_server lizmap_server
sudo rm -r qgis-lizmap-server-plugin-2.9.0
```
Modificare le variabili di ambiente
```
sudo nano /etc/apache2/sites-available/qgis-server.conf
```
aggiungere
```
# ENABLE ENVIRONMENTALE VARIABLE FOR LIZMAP SERVER
FcgidInitialEnv QGIS_SERVER_LIZMAP_REVEAL_SETTINGS True
FcgidInitialEnv QGIS_PLUGINPATH "/usr/share/qgis/python/plugins"
```