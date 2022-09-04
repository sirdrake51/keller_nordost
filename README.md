# keller_nordost
tests für keller-druckmesstechnik
# Daten von Keller Messgeräten via NodeRed in InfluxDB speichern und via Grafana auswerten


## Vorraussetzungen
* Docker-Desktop https://docs.docker.com/desktop/
* Visualstudio Code https://code.visualstudio.com/docs/setup/setup-overview
* InfluxDB 1.8 (als container)
* NodeRed 3 (als container)
* ThingsNetwork V3 Account und dort befindliche Keller Geräte

Getested wurde alles unter MacOs 12.

## Setup
* Installation von Docker-Desktop, Extension Portainer installiern 
![6097a7b03b7f8a144993184cf03aab5f.png](resources/541988e0bbf8459c8d8b30c7ee910a0c.png)
![Docker-Portainer](https://github.com/sirdrake51/keller_nordost/docker-portainer-extension.png)
* Installation von Visualstudio Code, Erweiterung für Docker installieren
* GitHub Verzeichnis herunter laden (cloning)

### Configuration

Zunächst das Githab Respository in ein lokales Verzeichnis clonen.
Das lokale Verzeichnis mit VisualStudio Code öffnen.
![e9cf64b9eda37c85a0c79964bc7fe3bb.png](resources/d398e7456139482f9e54c49901838b29.png)
![Visual-Studio](https://github.com/sirdrake51/keller_nordost/visualstudio.png)

Docker-Desktop öffnen und im Plugin Portainer ein neues Netzwerk mit dem Namen Keller anlegen, da wird für die Kommunikation der Container untereinander benötigt.
(Bild)
![ca53be06ccd9300ff8eec4ed56152c7b.png](resources/d8d1cc4d01554a1a9613fef1dc95f932.png)
![Portainer-Network](https://github.com/sirdrake51/keller_nordost/portainer-network.png)

**NodeRed**
In VisualStudio Code die Datei „ Nodered“ mit Docker starten.
(Bild)
![018665c4c260dc87b090cff4b585049f.png](resources/642a9e5f087140d4b630f8d445036f0f.png)
![VisualStudio-compose](https://github.com/sirdrake51/keller_nordost/visualstudio-compose.png)
Im Webbrowser sollte unter [nodered](http://localhost:1880) die Weboberfläche von NodeRed gestartet sein.
![8550ab426ad7600ef6673c816b5bf28c.png](resources/42d96452cdf54f49b8176e863dac5fec.png)
![Nodered-Menu](https://github.com/sirdrake51/keller_nordost/nodered/nodered-menu.png)

In NodeRed nun unter dem „drei Strich Menü" den Flow aus dem GitHub Verzeichnis importieren.
Nach dem Import erscheint die Fehlermeldung des nicht installierten NodeRed Plugins „influx“.  Das sollte noch nach installiert werden, unter Pallette verwalten.
(Bild)
![Influx-Plugin](https://github.com/sirdrake51/measuring_from_keller_pressure/nodered-install-influxdb.png)

Danach müssen für TTN und für Influx die korrekten Verbindungen her gerichtet werden. Dies umfasst die User und Passwörter nebst der Token (ja nach APP). Siehe dazu auch
[TTN-NodeRed](https://www.thethingsindustries.com/docs/integrations/node-red/)
Bei der IP-Adresse der Influx Datenbank muss die verwendet werden, die der influx Container nutzt. Demnach ist dies nicht etwa localhost sondern beispielsweise 172.19.0.2 - diese Adrasse sieht man am besten in der Containeransicht von Portainer.
Ist das erledigt, kann mit "deploy" die Änderungen im Flow gesichert werden und dieser wird neu gestartet. Mittels debug kann man dann kontrollieren, ob der Datentransfer funktioniert.

**Influx_DB**

In VisualStudio Code die Datei „ InfluxDB“ mit Docker starten.
Ist der Contaier erfolgreich hoch gefahren, kann man sich mit den Daten aus der .env in Grafana einloggen. Dort sollte als erstes die Verbindung zur Datenbank hergestellt werden.
(Bild)
![Grafana-Datenbank-Connect](https://github.com/sirdrake51/measuring_from_keller_pressure/grafana/grafana-source-connect.png)

Die Adresse sollte die gleiche sein wie unter NodeRed eingestellt. Der Datanbankname, der User und das Passwort kommen wieder aus der .env.
Anmerkung: sollte localhost nicht funktionieren, dann hat es mit der IP des Containers von Influx funktioniert. Die erhält man in der Ansciht der Container des Docker-Plugins Portainer
das war es, sofern nun die Daten von TTN eintreffen, sollten diese auch in der Datenbank ankommen.
zum Test befindet sich in NodeRed ein Flow, der Daten in die Influx-DB schreibt und liest. Mittels Debug innerhalb von NodeRed kann kontrolliert werden, ob alles korrekt funktioniert.
Als zweiten Test mal im Portainer auf das Symbol links neben dem Stecker klicken. In dem sich öffnenden Fenster wählt man bin/sh und dann "connect" und befindet sich in der Container Shell der influx_db.
Dort gibt man im Terminal nacheinander die Dinge in ein. Werden unter show measurements Testdaten angezeigt, so wird die Datenbank korrekt befüllt, es sind die Daten die aus NodeRed kommen.

```
influx
```
darauf sollte die Meldung kommen
``
Connected to http://localhost:8086 version 1.8.10
InfluxDB shell version: 1.8.10
``
weiter geht es mit der Eingabe
```
show databases

```
es sollte dann erscheinen
``
name: databases
name
keller_db
_internal
``
..dann schauen wir mal in die Datenbank
```
use keller_db

```
``
Using database keller_db
``
und schließlich
```
show measurements

```
``
name: measurements
name
Testdaten
``
Damit können die Daten aus dem Things Network in die Datenbank fließen.

**Grafana**
Passend zu den Keller Geräten (hier ADT1) sind im Verzeichniss enige Beispieldashboard und die Werte aus der influxdb zu viaualisieren. Achtung auch hier müssen die Werte mit denen angepasst werden, die aus der InfluxDB kommen.
Man kann diese in der Dasboardanschicht einfach importieren. Eines, das Log der Spannungen sieht zum Beispiel so aus.
![488c093fd94167626806b5f344fc8548.png](resources/e037854830c94351ad64c216c24dab91.png)

![Grafana-Beispiel-Dashboard](https://github.com/sirdrake51/measuring_from_keller_pressure/grafana/grafana-dashboard-adt-volt.png)
Hinweis dazu: bei diesen Templates müssen aber die Abfragen komplett neu aus der eigenen InfluxDB aufgebaut werden. 


