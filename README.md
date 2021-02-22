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

```xml
<?xml version="1.0" encoding="utf-8"?>
<WMS_Capabilities xmlns:qgs="http://www.qgis.org/wms" version="1.3.0" xsi:schemaLocation="http://www.opengis.net/wms http://schemas.opengis.net/wms/1.3.0/capabilities_1_3_0.xsd http://www.opengis.net/sld http://schemas.opengis.net/sld/1.1.0/sld_capabilities.xsd http://www.qgis.org/wms http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi?SERVICE=WMS&amp;REQUEST=GetSchemaExtension" xmlns:sld="http://www.opengis.net/sld" xmlns="http://www.opengis.net/wms" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
 <Service>
  <Name>WMS</Name>
  <Title>untitled</Title>
  <KeywordList>
   <Keyword vocabulary="ISO">infoMapAccessService</Keyword>
  </KeywordList>
  <OnlineResource xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi" xlink:type="simple"/>
  <Fees>None</Fees>
  <AccessConstraints>None</AccessConstraints>
 </Service>
 <Capability>
  <Request>
   <GetCapabilities>
    <Format>text/xml</Format>
    <DCPType>
     <HTTP>
      <Get>
       <OnlineResource xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi?" xlink:type="simple"/>
      </Get>
     </HTTP>
    </DCPType>
   </GetCapabilities>
   <GetMap>
    <Format>image/jpeg</Format>
    <Format>image/png</Format>
    <Format>image/png; mode=16bit</Format>
    <Format>image/png; mode=8bit</Format>
    <Format>image/png; mode=1bit</Format>
    <Format>application/dxf</Format>
    <DCPType>
     <HTTP>
      <Get>
       <OnlineResource xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi?" xlink:type="simple"/>
      </Get>
     </HTTP>
    </DCPType>
   </GetMap>
   <GetFeatureInfo>
    <Format>text/plain</Format>
    <Format>text/html</Format>
    <Format>text/xml</Format>
    <Format>application/vnd.ogc.gml</Format>
    <Format>application/vnd.ogc.gml/3.1.1</Format>
    <Format>application/json</Format>
    <Format>application/geo+json</Format>
    <DCPType>
     <HTTP>
      <Get>
       <OnlineResource xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi?" xlink:type="simple"/>
      </Get>
     </HTTP>
    </DCPType>
   </GetFeatureInfo>
   <sld:GetLegendGraphic>
    <Format>image/jpeg</Format>
    <Format>image/png</Format>
    <DCPType>
     <HTTP>
      <Get>
       <OnlineResource xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi?" xlink:type="simple"/>
      </Get>
     </HTTP>
    </DCPType>
   </sld:GetLegendGraphic>
   <sld:DescribeLayer>
    <Format>text/xml</Format>
    <DCPType>
     <HTTP>
      <Get>
       <OnlineResource xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi?" xlink:type="simple"/>
      </Get>
     </HTTP>
    </DCPType>
   </sld:DescribeLayer>
   <qgs:GetStyles>
    <Format>text/xml</Format>
    <DCPType>
     <HTTP>
      <Get>
       <OnlineResource xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="http://143.198.3.27/cgi-bin/qgis_mapserv.fcgi?" xlink:type="simple"/>
      </Get>
     </HTTP>
    </DCPType>
   </qgs:GetStyles>
  </Request>
  <Exception>
   <Format>XML</Format>
  </Exception>
  <sld:UserDefinedSymbolization SupportSLD="1" RemoteWCS="0" UserLayer="0" UserStyle="1" RemoteWFS="0" InlineFeature="0"/>
  <Layer queryable="0">
   <KeywordList>
    <Keyword vocabulary="ISO">infoMapAccessService</Keyword>
   </KeywordList>
   <CRS>CRS:84</CRS>
   <CRS>EPSG:4326</CRS>
   <CRS>EPSG:3857</CRS>
   <EX_GeographicBoundingBox>
    <westBoundLongitude>-0.000001</westBoundLongitude>
    <eastBoundLongitude>0.000001</eastBoundLongitude>
    <southBoundLatitude>-0.000001</southBoundLatitude>
    <northBoundLatitude>0.000001</northBoundLatitude>
   </EX_GeographicBoundingBox>
   <BoundingBox maxx="0.001" miny="-0.001" maxy="0.001" minx="-0.001" CRS="EPSG:3857"/>
   <BoundingBox maxx="0.000001" miny="-0.000001" maxy="0.000001" minx="-0.000001" CRS="EPSG:4326"/>
  </Layer>
 </Capability>
</WMS_Capabilities>
```

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


