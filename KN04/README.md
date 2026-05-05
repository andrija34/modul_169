# KN04 – Containers in Action & Orchestration

## Einführung

In diesem Kompetenznachweis wurden verschiedene Docker- und Docker-Swarm-Technologien praktisch umgesetzt. Dabei wurden Container erstellt, verwaltet, orchestriert und skaliert. Zusätzlich wurde ein Docker-Swarm-Cluster mit mehreren Nodes aufgebaut.

Die Arbeiten wurden auf AWS EC2 Instanzen durchgeführt.

---

## A) Docker Image erstellen, Registry und Deployment

### Ziel

In diesem Teil wurde ein eigenes Docker Image erstellt, angepasst und in eine Container Registry hochgeladen. Danach wurde die Anwendung auf einer AWS EC2 Instanz gestartet.

---

### Repository klonen

```bash
git clone <repo-url>
cd container-bootstrap/01_container
```

**Beschreibung:** Mit diesem Befehl wurde das vorbereitete Repository lokal auf die EC2 Instanz heruntergeladen.

📸 `images/kn04-a-git-clone.png`

---

### Dateien anpassen

#### main.css

Die Hintergrundfarbe wurde angepasst.

**Alt:**
```css
background-color: purple;
```

**Neu:**
```css
background-color: #ebd63d;
```

**Beschreibung:** Die Webseite musste gemäss Auftrag angepasst werden.

📸 `images/kn04-a-main-css.png`

---

### Bild ersetzen

```bash
rm image.png
mv image-new.png image.png
```

**Beschreibung:** Das alte Cloud-Native Bild wurde durch das Modul-169 Bild ersetzt.

📸 `images/kn04-a-image-replace.png`

---

### Port ändern

**Datei:** `app.js`

```js
const PORT = 8091;
```

**Beschreibung:** Die Anwendung wurde auf Port 8091 angepasst.

📸 `images/kn04-a-port-change.png`

---

### Dockerfile anpassen

```dockerfile
EXPOSE 8091
```

**Beschreibung:** Der neue Port musste auch im Dockerfile definiert werden.

📸 `images/kn04-a-dockerfile.png`

---

### Docker Image bauen

```bash
docker build -t ghcr.io/andrija34/webapp_andrija34_8091:1.0 .
```

**Beschreibung:** Mit diesem Befehl wurde das Docker Image erstellt.

📸 `images/kn04-a-build.png`

---

### ⚠️ Fehler: Falsche Registry

**Fehler:** `registry.github.com`

**Problem:** Es wurde zuerst die falsche Registry verwendet.

**Lösung:** Verwendung von `ghcr.io`

📸 `images/kn04-a-registry-error.png`

---

### Docker Login

```bash
docker login ghcr.io
```

**Beschreibung:** Login in die GitHub Container Registry.

📸 `images/kn04-a-login.png`

---

### Docker Push

```bash
docker push ghcr.io/andrija34/webapp_andrija34_8091:1.0
```

**Beschreibung:** Das Docker Image wurde in die Registry hochgeladen.

📸 `images/kn04-a-push.png`

---

### Container starten

```bash
docker run -d -p 8091:8091 ghcr.io/andrija34/webapp_andrija34_8091:1.0
```

**Beschreibung:** Die Anwendung wurde als Container gestartet.

📸 `images/kn04-a-run.png`

---

### ⚠️ Fehler: Website nicht erreichbar

**Fehler:** `ERR_CONNECTION_TIMED_OUT`

**Ursache:** Port 8091 war nicht in der AWS Security Group freigegeben.

**Lösung:** Port 8091 wurde in den Inbound Rules freigegeben.

📸 `images/kn04-a-timeout-error.png`

---

### Erfolgreicher Zugriff

Die Webseite war danach erreichbar unter `http://PUBLIC-IP:8091`

📸 `images/kn04-a-working-website.png`

---

## B) Docker Compose

### Ziel

In diesem Teil wurde eine Multi-Container Anwendung mit Docker Compose aufgebaut.

---

### Compose Datei anpassen

**Datei:** `compose.yml`

**Port ändern:**
```yaml
published: 5169
target: 8080
```

**Netzwerk ändern:**
```yaml
mil-net
```

**Volume ändern:**
```yaml
mil-vol
```

---

### ⚠️ Fehler: Undefined Network

**Fehler:** `service "web-fe" refers to undefined network mil-net`

**Ursache:** Das Netzwerk war oben falsch definiert.

**Lösung:** Das Netzwerk wurde korrekt ergänzt:

```yaml
networks:
  mil-net:
```

📸 `images/kn04-b-network-error.png`

---

### Compose starten

```bash
docker compose up -d
```

📸 `images/kn04-b-compose-up.png`

---

### Compose neu bauen

```bash
docker compose down
docker compose up -d --build
```

**Beschreibung:** Nach Änderungen musste die Umgebung neu gebaut werden.

📸 `images/kn04-b-compose-rebuild.png`

---

### Website testen

`http://PUBLIC-IP:5169`

📸 `images/kn04-b-working-site.png`

---

## C) Docker Swarm Cluster

### Ziel

Es wurde ein Docker Swarm Cluster mit 5 Nodes aufgebaut.

---

### Architektur

| Node  | Rolle   |
|-------|---------|
| Node1 | Manager |
| Node2 | Manager |
| Node3 | Manager |
| Node4 | Worker  |
| Node5 | Worker  |

📸 `images/kn04-c-architecture.png`

---

### Swarm initialisieren

```bash
sudo docker swarm init --advertise-addr 172.31.39.123
```

📸 `images/kn04-c-swarm-init.png`

---

### ⚠️ Fehler: Permission denied

**Fehler:** `permission denied while trying to connect to the docker API`

**Ursache:** Docker wurde ohne `sudo` ausgeführt.

**Lösung:**

```bash
sudo docker swarm init ...
```

📸 `images/kn04-c-permission-error.png`

---

### Join Token holen

**Manager:**

```bash
sudo docker swarm join-token manager
```

**Worker:**

```bash
sudo docker swarm join-token worker
docker swarm join --token ...
```

**Beschreibung:** Die übrigen Nodes wurden dem Cluster hinzugefügt.

📸 `images/kn04-c-join-nodes.png`

---

### Cluster prüfen

```bash
sudo docker node ls
```

**Beschreibung:** Alle 5 Nodes wurden erfolgreich angezeigt.

📸 `images/kn04-c-node-ls.png`

---

### Manager auf Drain setzen

```bash
sudo docker node update --availability drain <NODE>
```

**Beschreibung:** Damit keine Container auf den Manager Nodes laufen.

📸 `images/kn04-c-drain.png`

---

## D) Docker Swarm Imperativ

### Service erstellen

```bash
sudo docker service create \
  --name 169-web \
  --publish 5010:80 \
  --replicas 5 \
  nginx
```

📸 `images/kn04-d-service-create.png`

---

### Service prüfen

```bash
sudo docker service ps 169-web
```

📸 `images/kn04-d-service-ps.png`

---

### Service skalieren

```bash
sudo docker service scale 169-web=10
```

📸 `images/kn04-d-scale.png`

---

### Container löschen

```bash
sudo docker container rm -f <id>
```

**Beschreibung:** Mehrere Container wurden absichtlich gelöscht.

📸 `images/kn04-d-container-delete.png`

---

### Self-Healing

**Beschreibung:** Docker Swarm hat automatisch neue Container erstellt.

**Fehler:** `task: non-zero exit (137)`

**Bedeutung:** Container wurde manuell beendet.

📸 `images/kn04-d-self-healing.png`

---

## E) Docker Swarm Deklarativ

### compose.yml anpassen

**Änderungen:**

```yaml
replicas: 6
published: 5169
target: 8169
```

📸 `images/kn04-e-compose-edit.png`

---

### app.py anpassen

```python
app.run(host="0.0.0.0", port=8169)
```

📸 `images/kn04-e-app-py.png`

---

### Dockerfile anpassen

```dockerfile
EXPOSE 8169
```

📸 `images/kn04-e-dockerfile.png`

---

### Image bauen

```bash
docker build -t ghcr.io/andrija34/modul_169-swarm-stack:1.0 .
```

📸 `images/kn04-e-build.png`

---

### Push

```bash
docker push ghcr.io/andrija34/modul_169-swarm-stack:1.0
```

📸 `images/kn04-e-push.png`

---

### ⚠️ Fehler: docker push requires 1 argument

**Ursache:** Es wurde fälschlicherweise ein Punkt `.` verwendet.

**Falsch:**
```bash
docker push ghcr.io/... .
```

**Richtig:**
```bash
docker push ghcr.io/...
```

📸 `images/kn04-e-push-error.png`

---

### Stack deployen

```bash
sudo docker stack deploy -c compose.yml mil-zaehler
```

📸 `images/kn04-e-stack-deploy.png`

---

### ⚠️ Fehler: not a swarm manager

**Ursache:** Der Befehl wurde auf einem Worker Node ausgeführt.

**Lösung:** Deploy auf dem Leader Manager Node durchführen.

📸 `images/kn04-e-manager-error.png`

---

### ⚠️ Fehler: No such image

**Ursache:** Die Worker Nodes konnten das GitHub Image nicht herunterladen.

**Lösung:** Docker Login auf den Worker Nodes:

```bash
sudo docker login ghcr.io
```

📸 `images/kn04-e-no-image-error.png`

---

### Stack prüfen

```bash
sudo docker stack services mil-zaehler
sudo docker stack ps mil-zaehler
```

📸 `images/kn04-e-stack-ps.png`

---

### Skalierung ändern

```yaml
replicas: 4
```

**Beschreibung:** Die Anzahl Replicas wurde reduziert.

📸 `images/kn04-e-scale-down.png`

---

### Node-Ausfall simulieren

Ein Worker Node wurde in AWS terminiert.

**Beschreibung:** Docker Swarm hat die Container automatisch auf den verbleibenden Worker verschoben.

📸 `images/kn04-e-node-failure.png`

---

### Stack löschen

```bash
sudo docker stack rm mil-zaehler
```

📸 `images/kn04-e-stack-remove.png`

---

## Fazit

In diesem Kompetenznachweis wurden Docker, Docker Compose und Docker Swarm praktisch umgesetzt. Es wurde gezeigt, wie Container erstellt, skaliert und orchestriert werden können. Zusätzlich wurden High Availability, Self-Healing und deklarative Deployments getestet.
