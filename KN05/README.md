# KN05 – Container Monitoring & Security

## Einführung

In diesem Kompetenznachweis wurden verschiedene Technologien für Logging, Monitoring und Sicherheit von Containern umgesetzt.

Dabei wurden:

* Container-Logs analysiert
* Syslog verwendet
* Monitoring mit Prometheus, Grafana und cAdvisor aufgebaut
* Docker Compose eingesetzt
* Sicherheitsmechanismen wie Trivy, AppArmor und seccomp getestet

Die Arbeiten wurden auf einer AWS EC2 Ubuntu Instanz durchgeführt.

---

## A) Logging von Containern

### Ausgangslage

Docker-Container schreiben ihre Logs standardmässig auf STDOUT und STDERR.
Mit dem Befehl `docker logs` können diese Logs angezeigt werden.
Zusätzlich kann Docker Logs direkt an das Syslog des Hosts weiterleiten.

---

### Schritt 1 – Container starten und Logs anzeigen

#### Container starten

```bash
docker run --name m169_logtest ubuntu bash -c 'echo "Hallo"; echo "Fehler" >&2'
```

**Beschreibung:** Der Container wurde gestartet und erzeugte eine normale Ausgabe sowie eine Fehlermeldung.

📸 `images/kn05-a-container-start.png`

---

#### Logs anzeigen

```bash
docker logs m169_logtest
```

**Beschreibung:** Mit diesem Befehl wurden die Logs des Containers angezeigt.

📸 `images/kn05-a-docker-logs.png`

---

#### Container löschen

```bash
docker rm m169_logtest
```

**Beschreibung:** Der gestoppte Container wurde entfernt.

📸 `images/kn05-a-container-delete.png`

---

### Schritt 2 – Laufende Logs überwachen

#### Container im Hintergrund starten

```bash
docker run -d --name m169_logtest ubuntu bash -c 'while true; do echo "tick"; sleep 1; done;'
```

**Beschreibung:** Der Container wurde im Hintergrund gestartet und erzeugte jede Sekunde eine Ausgabe.

📸 `images/kn05-a-running-container.png`

---

#### Logs anzeigen

```bash
docker logs m169_logtest
```

#### Logs live anzeigen

```bash
docker logs -f m169_logtest
```

**Beschreibung:** Mit `-f` wurden die Logs live überwacht.

📸 `images/kn05-a-live-logs.png`

---

#### Anzahl Log-Zeilen zählen

```bash
docker logs m169_logtest | wc -l
```

**Beschreibung:** Mit diesem Befehl wurde die Anzahl Log-Zeilen gezählt.

📸 `images/kn05-a-log-count.png`

---

#### Container stoppen und löschen

```bash
docker stop m169_logtest
```

```bash
docker rm m169_logtest
```

**Beschreibung:** Der Container wurde zuerst gestoppt und danach gelöscht.

📸 `images/kn05-a-stop-remove.png`

---

### Schritt 3 – Syslog konfigurieren

#### Container mit Syslog starten

```bash
docker run -d --log-driver=syslog ubuntu bash -c 'i=0; while true; do i=$((i+1)); echo "docker $i"; sleep 1; done;'
```

**Beschreibung:** Die Logs wurden direkt in das Syslog des Hosts geschrieben.

📸 `images/kn05-a-syslog-container.png`

---

#### Syslog überwachen

```bash
tail -f /var/log/syslog
```

**Beschreibung:** Mit diesem Befehl wurden die Logs in Echtzeit überwacht.

📸 `images/kn05-a-tail-syslog.png`

---

### Erkenntnisse Logging

* Docker speichert Logs standardmässig lokal.
* Logs können mit `docker logs` angezeigt werden.
* Mit `-f` können Logs live überwacht werden.
* Syslog ermöglicht zentrales Logging.

---

## B) Monitoring und Reporting von Containern

### Ausgangslage

Für das Monitoring wurde eine Umgebung mit folgenden Diensten aufgebaut:

* Prometheus
* Grafana
* cAdvisor
* Alertmanager
* Node Exporter

Die Umgebung wurde deklarativ mit Docker Compose erstellt.

---

### Schritt 1 – Repository klonen

#### Repository klonen

```bash
git clone https://gitlab.com/ser-cal/m169-scripts.git
```

#### Verzeichnis wechseln

```bash
cd m169-scripts/KN05_B
```

**Beschreibung:** Das bestehende Repository wurde geklont und das passende Verzeichnis geöffnet.

📸 `images/kn05-b-git-clone.png`

---

### ⚠️ Fehler – docker compose funktionierte nicht

**Fehlermeldung:**

```text
unknown shorthand flag: 'd' in -d
```

**Beschreibung:** Die installierte Docker-Version unterstützte den Befehl `docker compose` nicht.

📸 `images/kn05-b-docker-compose-error.png`

---

### Lösung – docker-compose installieren

#### Installation

```bash
sudo apt update
```

```bash
sudo apt install docker-compose -y
```

#### Version prüfen

```bash
docker-compose --version
```

**Beschreibung:** Docker Compose wurde nachinstalliert.

📸 `images/kn05-b-install-compose.png`

---

### Docker Compose starten

```bash
docker-compose up -d
```

**Beschreibung:** Die komplette Monitoring-Umgebung wurde gestartet.

📸 `images/kn05-b-compose-up.png`

---

### Schritt 2 – Dienste prüfen

#### Grafana

```text
http://<EC2-Public-IP>:3000
```

📸 `images/kn05-b-grafana.png`

---

#### cAdvisor

```text
http://<EC2-Public-IP>:8080
```

📸 `images/kn05-b-cadvisor.png`

---

#### Prometheus

```text
http://<EC2-Public-IP>:9090
```

📸 `images/kn05-b-prometheus.png`

---

### Schritt 3 – Dashboard importieren

#### Login

**Benutzer:** `admin`

**Passwort:** `admin`

---

#### Dashboard importieren

**Dashboard-ID:** `21361`

**Beschreibung:** Das Dashboard 21361 wurde importiert und mit Prometheus verbunden.

📸 `images/kn05-b-dashboard-import.png`

---

#### Metriken anzeigen

Im Dashboard wurden folgende Metriken angezeigt:

* CPU Usage
* Memory Usage
* Netzwerk
* Container Status

📸 `images/kn05-b-dashboard-metrics.png`

---

### Schritt 4 – Stresstest durchführen

#### Stress-Test starten

```bash
docker run --rm --name stress-test ubuntu:latest bash -c "apt update && apt install -y stress && stress --cpu 2 --timeout 60s"
```

**Beschreibung:** Der Container erzeugte CPU-Last für 60 Sekunden.

📸 `images/kn05-b-stress-test.png`

---

#### Auswirkungen in Grafana

**Beschreibung:** Während des Stresstests stiegen CPU- und RAM-Auslastung deutlich an.

📸 `images/kn05-b-grafana-stress.png`

---

### Schritt 5 – Umgebung löschen

#### Umgebung stoppen

```bash
docker-compose down -v
```

**Beschreibung:** Alle Container und Volumes der Monitoring-Umgebung wurden entfernt.

📸 `images/kn05-b-compose-down.png`

---

### Erkenntnisse Monitoring

* Docker Compose vereinfacht komplexe Setups.
* Grafana visualisiert Metriken übersichtlich.
* Prometheus sammelt Metriken zuverlässig.
* Monitoring hilft Probleme frühzeitig zu erkennen.

---

## C) Sicherheit von Containern

### Ausgangslage

Container bringen Sicherheitsrisiken mit sich:

* Unsichere Images
* Zu hohe Berechtigungen
* Container Breakouts
* Kernel Exploits

Zur Verbesserung der Sicherheit wurden verschiedene Schutzmechanismen getestet.

---

### Schritt 1 – Trivy installieren

#### ⚠️ Fehler bei Installation

```text
Error: Unable to locate package trivy
```

**Beschreibung:** Das Paket war nicht im Standard-Repository vorhanden.

📸 `images/kn05-c-trivy-error.png`

---

### Lösung – Repository hinzufügen

#### Benötigte Pakete installieren

```bash
sudo apt update
```

```bash
sudo apt install wget apt-transport-https gnupg lsb-release -y
```

#### GPG-Key hinzufügen

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
```

#### Repository hinzufügen

```bash
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
```

#### Trivy installieren

```bash
sudo apt update
```

```bash
sudo apt install trivy -y
```

📸 `images/kn05-c-install-trivy.png`

---

### ⚠️ Fehler – Speicherplatzproblem bei Trivy

```text
Fatal error: disk quota exceeded
```

**Beschreibung:** Während des Java-DB Downloads war zu wenig Speicherplatz vorhanden.

📸 `images/kn05-c-trivy-disk-error.png`

---

### Lösung – Nur Vulnerability Scanner verwenden

```bash
trivy image --scanners vuln nginx:latest
```

**Beschreibung:** Der Scan wurde ohne Secret- und Java-DB durchgeführt.

📸 `images/kn05-c-trivy-scan.png`

---

### Schritt 2 – Container mit minimalen Rechten starten

#### Ohne Root starten

```bash
docker run --user 1000:1000 nginx
```

**Beschreibung:** Der Container wurde ohne Root-Rechte gestartet.

📸 `images/kn05-c-non-root.png`

---

#### Read-Only Filesystem

```bash
docker run --read-only --tmpfs /tmp:rw,size=64M nginx
```

**Beschreibung:** Das Dateisystem wurde schreibgeschützt gestartet.

📸 `images/kn05-c-readonly.png`

---

### Schritt 3 – AppArmor und seccomp

#### ⚠️ Fehler bei seccomp

```text
docker: opening seccomp profile (default.json) failed: open default.json: no such file or directory
```

**Beschreibung:** Die seccomp-Konfigurationsdatei war nicht vorhanden.

📸 `images/kn05-c-seccomp-error.png`

---

### Lösung – AppArmor verwenden

```bash
sudo docker run --security-opt apparmor=docker-default nginx
```

**Beschreibung:** Der Container wurde mit dem Standard-AppArmor-Profil gestartet.

📸 `images/kn05-c-apparmor.png`

---

### Erkenntnisse Sicherheit

* Trivy erkennt Schwachstellen in Images.
* Container sollten nicht als Root laufen.
* Read-Only Filesysteme erhöhen die Sicherheit.
* AppArmor begrenzt Container-Zugriffe.

---

## Fazit

In diesem Kompetenznachweis wurden Logging-, Monitoring- und Sicherheitsfunktionen für Container erfolgreich umgesetzt.

Wichtige Erkenntnisse:

* Docker-Logs können lokal oder zentral gespeichert werden.
* Monitoring mit Prometheus und Grafana erleichtert die Überwachung.
* Docker Compose vereinfacht komplexe Umgebungen.
* Sicherheitsmechanismen wie Trivy und AppArmor erhöhen die Sicherheit von Containern.

---

## Quellenverzeichnis

* Docker Dokumentation
* Docker Compose Dokumentation
* Prometheus Dokumentation
* Grafana Dokumentation
* Trivy Dokumentation
* cAdvisor GitHub Repository
