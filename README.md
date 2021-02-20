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
sudo apt install qgis-server
sudo apt install python-qgis
```

L'eseguibile di QGIS server è il file ```qgis_mapserv.fcgi```. Il file dovrebbe essere presente nel percorso /usr/lib/cgi-bin/qgis_mapserv.fcgi.

QGIS serve necessita di un X server per essere completamente funzionante (ad esempio la funazione getprint).

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


