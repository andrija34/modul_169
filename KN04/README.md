```
# KN04 -- Containers in Action & Orchestration# EinführungIn diesem Kompetenznachweis wurden verschiedene Docker- und Docker-Swarm-Technologien praktisch umgesetzt.  Dabei wurden Container erstellt, verwaltet, orchestriert und skaliert. Zusätzlich wurde ein Docker-Swarm-Cluster mit mehreren Nodes aufgebaut.Die Arbeiten wurden auf AWS EC2 Instanzen durchgeführt.---# A) Docker Image erstellen, Registry und Deployment## ZielIn diesem Teil wurde ein eigenes Docker Image erstellt, angepasst und in eine Container Registry hochgeladen. Danach wurde die Anwendung auf einer AWS EC2 Instanz gestartet.---# Repository klonenZuerst wurde das Repository geklont.```bashgit clone <repo-url>cd container-bootstrap/01_container
```

Beschreibung
------------

Mit diesem Befehl wurde das vorbereitete Repository lokal auf die EC2 Instanz heruntergeladen.

Screenshot
----------

```
images/kn04-a-git-clone.png
```

* * * * *

Dateien anpassen
================

main.css
--------

Die Hintergrundfarbe wurde angepasst.

Alt:

```
background-color: purple;
```

Neu:

```
background-color: #ebd63d;
```

Beschreibung
------------

Die Webseite musste gemäss Auftrag angepasst werden.

Screenshot
----------

```
images/kn04-a-main-css.png
```

* * * * *

Bild ersetzen
=============

Das Bild `image.png` wurde ersetzt.

```
rm image.pngmv image-new.png image.png
```

Beschreibung
------------

Das alte Cloud-Native Bild wurde durch das Modul-169 Bild ersetzt.

Screenshot
----------

```
images/kn04-a-image-replace.png
```

* * * * *

Port ändern
===========

Die Anwendung wurde auf Port 8091 angepasst.

Datei:

```
app.js
```

Änderung:

```
const PORT = 8091;
```

Screenshot
----------

```
images/kn04-a-port-change.png
```

* * * * *

Dockerfile anpassen
===================

```
EXPOSE 8091
```

Beschreibung
------------

Der neue Port musste auch im Dockerfile definiert werden.

Screenshot
----------

```
images/kn04-a-dockerfile.png
```

* * * * *

Docker Image bauen
==================

```
docker build -t ghcr.io/andrija34/webapp_andrija34_8091:1.0 .
```

Beschreibung
------------

Mit diesem Befehl wurde das Docker Image erstellt.

Screenshot
----------

```
images/kn04-a-build.png
```

* * * * *

Fehler: Falsche Registry
========================

Fehler:

```
registry.github.com
```

Problem
-------

Es wurde zuerst die falsche Registry verwendet.

Lösung
------

Verwendung von:

```
ghcr.io
```

Screenshot
----------

```
images/kn04-a-registry-error.png
```

* * * * *

Docker Login
============

```
docker login ghcr.io
```

Beschreibung
------------

Login in die GitHub Container Registry.

Screenshot
----------

```
images/kn04-a-login.png
```

* * * * *

Docker Push
===========

```
docker push ghcr.io/andrija34/webapp_andrija34_8091:1.0
```

Beschreibung
------------

Das Docker Image wurde in die Registry hochgeladen.

Screenshot
----------

```
images/kn04-a-push.png
```

* * * * *

Container starten
=================

```
docker run -d -p 8091:8091 ghcr.io/andrija34/webapp_andrija34_8091:1.0
```

Beschreibung
------------

Die Anwendung wurde als Container gestartet.

Screenshot
----------

```
images/kn04-a-run.png
```

* * * * *

Fehler: Website nicht erreichbar
================================

Fehler:

```
ERR_CONNECTION_TIMED_OUT
```

Ursache
-------

Port 8091 war nicht in der AWS Security Group freigegeben.

Lösung
------

Port 8091 wurde in den Inbound Rules freigegeben.

Screenshot
----------

```
images/kn04-a-timeout-error.png
```

* * * * *

Erfolgreicher Zugriff
=====================

Die Webseite war danach erreichbar:

```
http://PUBLIC-IP:8091
```

Screenshot
----------

```
images/kn04-a-working-website.png
```

* * * * *

B) Docker Compose
=================

Ziel
====

In diesem Teil wurde eine Multi-Container Anwendung mit Docker Compose aufgebaut.

* * * * *

Compose Datei anpassen
======================

Datei:

```
compose.yml
```

* * * * *

Port ändern
===========

```
published: 5169target: 8080
```

* * * * *

Netzwerk ändern
===============

```
mil-net
```

* * * * *

Volume ändern
=============

```
mil-vol
```

* * * * *

Fehler: Undefined Network
=========================

Fehler:

```
service "web-fe" refers to undefined network mil-net
```

Ursache
-------

Das Netzwerk war oben falsch definiert.

Lösung
------

Das Netzwerk wurde korrekt ergänzt:

```
networks:  mil-net:
```

Screenshot
----------

```
images/kn04-b-network-error.png
```

* * * * *

Compose starten
===============

```
docker compose up -d
```

Screenshot
----------

```
images/kn04-b-compose-up.png
```

* * * * *

Compose neu bauen
=================

```
docker compose downdocker compose up -d --build
```

Beschreibung
------------

Nach Änderungen musste die Umgebung neu gebaut werden.

Screenshot
----------

```
images/kn04-b-compose-rebuild.png
```

* * * * *

Website testen
==============

```
http://PUBLIC-IP:5169
```

Screenshot
----------

```
images/kn04-b-working-site.png
```

* * * * *

C) Docker Swarm Cluster
=======================

Ziel
====

Es wurde ein Docker Swarm Cluster mit 5 Nodes aufgebaut.

* * * * *

Architektur
===========

| Node | Rolle |
| --- | --- |
| Node1 | Manager |
| Node2 | Manager |
| Node3 | Manager |
| Node4 | Worker |
| Node5 | Worker |

Screenshot
----------

```
images/kn04-c-architecture.png
```

* * * * *

Swarm initialisieren
====================

```
sudo docker swarm init --advertise-addr 172.31.39.123
```

Screenshot
----------

```
images/kn04-c-swarm-init.png
```

* * * * *

Fehler: Permission denied
=========================

Fehler:

```
permission denied while trying to connect to the docker API
```

Ursache
-------

Docker wurde ohne sudo ausgeführt.

Lösung
------

```
sudo docker swarm init ...
```

Screenshot
----------

```
images/kn04-c-permission-error.png
```

* * * * *

Join Token holen
================

Manager:

```
sudo docker swarm join-token manager
```

Worker:

```
sudo docker swarm join-token worker
```

Screenshot
----------

```
images/kn04-c-join-token.png
```

* * * * *

Nodes verbinden
===============

```
docker swarm join --token ...
```

Beschreibung
------------

Die übrigen Nodes wurden dem Cluster hinzugefügt.

Screenshot
----------

```
images/kn04-c-join-nodes.png
```

* * * * *

Cluster prüfen
==============

```
sudo docker node ls
```

Beschreibung
------------

Alle 5 Nodes wurden erfolgreich angezeigt.

Screenshot
----------

```
images/kn04-c-node-ls.png
```

* * * * *

Manager auf Drain setzen
========================

```
sudo docker node update --availability drain <NODE>
```

Beschreibung
------------

Damit keine Container auf den Manager Nodes laufen.

Screenshot
----------

```
images/kn04-c-drain.png
```

* * * * *

D) Docker Swarm Imperativ
=========================

Service erstellen
=================

```
sudo docker service create \--name 169-web \--publish 5010:80 \--replicas 5 \nginx
```

Screenshot
----------

```
images/kn04-d-service-create.png
```

* * * * *

Service prüfen
==============

```
sudo docker service ps 169-web
```

Screenshot
----------

```
images/kn04-d-service-ps.png
```

* * * * *

Service skalieren
=================

```
sudo docker service scale 169-web=10
```

Screenshot
----------

```
images/kn04-d-scale.png
```

* * * * *

Container löschen
=================

```
sudo docker container rm -f <id>
```

Beschreibung
------------

Mehrere Container wurden absichtlich gelöscht.

Screenshot
----------

```
images/kn04-d-container-delete.png
```

* * * * *

Self-Healing
============

Beschreibung
------------

Docker Swarm hat automatisch neue Container erstellt.

Fehler
------

```
task: non-zero exit (137)
```

Bedeutung
---------

Container wurde manuell beendet.

Screenshot
----------

```
images/kn04-d-self-healing.png
```

* * * * *

E) Docker Swarm deklarativ
==========================

compose.yml anpassen
====================

Änderungen
----------

```
replicas: 6published: 5169target: 8169
```

Screenshot
----------

```
images/kn04-e-compose-edit.png
```

* * * * *

app.py anpassen
===============

```
app.run(host="0.0.0.0", port=8169)
```

Screenshot
----------

```
images/kn04-e-app-py.png
```

* * * * *

Dockerfile anpassen
===================

```
EXPOSE 8169
```

Screenshot
----------

```
images/kn04-e-dockerfile.png
```

* * * * *

Image bauen
===========

```
docker build -t ghcr.io/andrija34/modul_169-swarm-stack:1.0 .
```

Screenshot
----------

```
images/kn04-e-build.png
```

* * * * *

Push
====

```
docker push ghcr.io/andrija34/modul_169-swarm-stack:1.0
```

Screenshot
----------

```
images/kn04-e-push.png
```

* * * * *

Fehler: docker push requires 1 argument
=======================================

Ursache
-------

Es wurde fälschlicherweise ein Punkt `.` verwendet.

Falsch:

```
docker push ghcr.io/... .
```

Richtig:

```
docker push ghcr.io/...
```

Screenshot
----------

```
images/kn04-e-push-error.png
```

* * * * *

Stack deployen
==============

```
sudo docker stack deploy -c compose.yml mil-zaehler
```

Screenshot
----------

```
images/kn04-e-stack-deploy.png
```

* * * * *

Fehler: not a swarm manager
===========================

Ursache
-------

Der Befehl wurde auf einem Worker Node ausgeführt.

Lösung
------

Deploy auf dem Leader Manager Node durchführen.

Screenshot
----------

```
images/kn04-e-manager-error.png
```

* * * * *

Fehler: No such image
=====================

Ursache
-------

Die Worker Nodes konnten das GitHub Image nicht herunterladen.

Lösung
------

Docker Login auf den Worker Nodes:

```
sudo docker login ghcr.io
```

Screenshot
----------

```
images/kn04-e-no-image-error.png
```

* * * * *

Stack prüfen
============

```
sudo docker stack services mil-zaehler
```

```
sudo docker stack ps mil-zaehler
```

Screenshot
----------

```
images/kn04-e-stack-ps.png
```

* * * * *

Skalierung ändern
=================

```
replicas: 4
```

Beschreibung
------------

Die Anzahl Replicas wurde reduziert.

Screenshot
----------

```
images/kn04-e-scale-down.png
```

* * * * *

Node-Ausfall simulieren
=======================

Ein Worker Node wurde in AWS terminiert.

Beschreibung
------------

Docker Swarm hat die Container automatisch auf den verbleibenden Worker verschoben.

Screenshot
----------

```
images/kn04-e-node-failure.png
```

* * * * *

Stack löschen
=============

```
sudo docker stack rm mil-zaehler
```

Screenshot
----------

```
images/kn04-e-stack-remove.png
```

* * * * *

Fazit
=====

In diesem Kompetenznachweis wurden Docker, Docker Compose und Docker Swarm praktisch umgesetzt.\
Es wurde gezeigt, wie Container erstellt, skaliert und orchestriert werden können. Zusätzlich wurden High Availability, Self-Healing und deklarative Deployments getestet.
