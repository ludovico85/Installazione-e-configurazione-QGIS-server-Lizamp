# Installazione-e-configurazione-Lizamp e QGIS server
Il repository contiene le istruzioni per l'installazione di QGIS server e Lizmap su server Ubuntu.

La documentazione ufficiale di Lizmap può essere consultata al [link](https://docs.lizmap.com/current/it/index.html)

Bisogna aver configurato il server e installato apache2 (postgresql opzionale) (si può consultare [il repo](https://github.com/ludovico85/Installazione-e-configurazione-di-postgresql-e-postgis-su-server-ubuntu-20.04/blob/master/README.md))

## Installazione di QGIS server
https://qgis.org/en/site/forusers/alldownloads.html#linux

Lizmap utilizza QGIS server per la distribuzione dei dati sottoforma di servizi OGC. Prerequisito fondamentale è l'installazione di QGIS server.

Per installare QGIS server prima è necessario installare alcuni tool:

```
sudo apt install gnupg software-properties-common
```

Successivamente è necessario installare la chiave di installazione QGIS dal repository:

```
wget -qO - https://qgis.org/downloads/qgis-2020.gpg.key | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/qgis-archive.gpg --import
```

```
sudo chmod a+r /etc/apt/trusted.gpg.d/qgis-archive.gpg
```

Adesso è necessatrio aggiungere il repository e aggiornare i pacchetti di ubuntu:

```
sudo add-apt-repository "deb https://qgis.org/ubuntu `lsb_release -c -s` main"
sudo apt update
```

Installare qgis-server e alcuni plugin:

```
sudo apt install qgis-server libapache2-mod-fcgid --no-install-recommends --no-install-suggests
sudo apt install python-qgis
```

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

Abilitare il modulo di apache fcgid e abilitare lo script serve-cgi-bin

```
sudo a2enmod fcgid
sudo a2enconf serve-cgi-bin
```

Riavviare apache2:

```
sudo service apache2 restart
```

Creare il servizio di configurazione di QGIS server:

```
sudo touch /etc/apache2/sites-available/qgis-server.conf
sudo nano /etc/apache2/sites-available/qgis-server.conf
```

Incollare il seguente contenuto:

```
<VirtualHost *:80>
    ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
    <Directory "/usr/lib/cgi-bin/">
    Options ExecCGI FollowSymLinks
    Require all granted
    AddHandler fcgid-script .fcgi
    </Directory>
</VirtualHost>
```

Aggiungere il script a2dissite per disabilitare il virtual host di default:

```
sudo a2dissite 000-default.conf
```

Ora abilitare il file di configurazione del virtual host di qgis-server:

```
sudo a2ensite qgis-server.conf
```

Verificare la correttezza della sintassi:

```
apachectl configtest
```

Attivare il virtual host:

```
sudo systemctl restart apache2
```

Testare la corretta installazione di QGIS server digitando nel browser:

```
http://myhost/cgi-bin/qgis_mapserv.fcgi?SERVICE=WMS&VERSION=1.3.0&REQUEST=GetCapabilities
```

La risposta dovrebbe essere simile all'immagine qui sotto.

![Alt text](/img/xml.png)

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
VERSION=3.4.0
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

Riavviare apache2

```
sudo systemctl restart apache2
```

Per verificare il funzionamento digitare nel browser http://my_host/lizmap

## Lizmap Extra

### Redirect
```
sudo ln -s /var/www/lizmap-web-client-release_3_0/lizmap/www/ /var/www/html/nome
```

### Geolocalizzazione

Se si vuole utilizzare la funzione di geolocalizzazione è necesario che lo scambio dei dati avvenga attraverso protoclli certificati e sicuri quali l'HTTPS. Bisogna quindi configurare apache2 affinchè possa utilizzare il protocollo. A questo [link](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04) viene riportata una possibile soluzione.

### Gestione dei dati tramite PostGIS

Se si bule utilizzare Postgreslq/PostGIS per la gestione dei dati, basta installareil seguente modulo (dopo aver installato postgresql e l'estensione postgis):

```
sudo apt-get install php7.4-pgsql
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
