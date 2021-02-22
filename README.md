# Installazione-e-configurazione-Lizamp
Il repository contiene le istruzioni per l'installazione di Lizmap su server Ubuntu

La documentazione ufficiale di Lizmap può essere consultata al link https://docs.lizmap.com/current/it/index.html

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
apt install qgis-server --no-install-recommends --no-install-suggests
sudo apt install python-qgis
```

L'eseguibile di QGIS server è il file ```qgis_mapserv.fcgi```. Il file dovrebbe essere presente nel percorso /usr/lib/cgi-bin/qgis_mapserv.fcgi.

*** NOTA BENE ***
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
sudo apt install qgis-server --no-install-recommends --no-install-suggests
sudo apt install python-qgis
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

Ora abilitare il file di configurazione del virtual host pgAdmin:

```
sudo a2ensite pgadmin4.conf
```

Verificare la correttezza della sintassi:

```apachectl configtest
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



QGIS server necessita di un X server per essere completamente funzionante (ad esempio la funazione getprint).

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


