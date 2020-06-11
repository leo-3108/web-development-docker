# Web Development - Docker für Entwicklungsumgebung

## Motivation

Entwicklungsumgebungen (nicht nur) für Web-Anwendungen haben im Team ebenso wie alleine bestimmte Herausforderungen:

* In der Regel gibt es mehr als 1 Projekt
* Unterschiedlichen Programmiersprachen
* Unterschiedliche (System-)Abhängigkeiten
* Unterschiedliche Betriebssysteme
* Unterschiedliche Dienste wie Datenbank, Cache, Web-Server, Mailversand, etc.
* Alles in unterschiedlichen Versionen

### Wunschliste an Entwicklungsumgebung

* **Umgebung sollte…**
* …einfach zu installieren sein
* …reproduzierbar zu installieren sein
* …Qualitätsanforderungen genügen
* …gut dokumentiert sein
* …leicht zugänglich sein
* …bei allen im Team identisch sein
* …der Produktionsumgebung entsprechen

### Optionen

* Ausführliche und immer aktuelle Dokumentation der Umgebung
* Entwicklung auf zentralem Server
* Zentrale Verwaltung aller Computer
* Computer pro Projekt 💸
* Virtuelle Maschinen
* Automatisierung der Umgebung

### Option "Virtuelle Maschinen"

* Zentrale Verwaltung und "ein Computer pro Projekt"
* Muss auch provisioniert werden
* Muss verteilt werden
* **Vorsicht:** Große Daten (nicht ins Repository möglich)
* **Vorsicht:** Brauchen viele Ressourcen

### Option "Automatisierung"

* "Ausführbare" Dokumentation
* Liegt als Quellcode vor und damit Teil der bisher kennengelernten Prozesse
* Ausführung zeitaufwendig für Beteiligte
* Immer noch: Unterschiedliche Betriebssystem und unkontrollierbare Einflüsse
* **Vorsicht:** Veränderung eines Systems über die Zeit

### Options "Container"

* _Best of both worlds_
* Definition des Containers in Code
* Leichtgewichtig in der Ausführung
* Keine Simulation von Hardware notwendig
* Isoliert
* Reproduzierbar
* Leicht zu verteilen
* Leicht "zusammenzustecken"

## "Getting Started"-App starten

Kurze Einführung in Docker basierend auf dem [offiziellen Docker "Getting Started"](https://github.com/docker/getting-started).

Neben dem Beispiel-Code kann parallel die "Getting Started" App von Docker gestartet werden:

```
docker run --rm -dp 80:80 docker/getting-started
```

Im Anschluss lässt sich das Tutorial im Browser unter http://localhost aufrufen.

## Wichtige Befehle

Im folgenden wollen wir die wesentlichen Befehle zum Arbeiten mit Docker als Entwicklungsumgebung festhalten:

### Ad-Hoc Linux Shell

Ubuntu Docker-Image als universelle Basis (Quelle für Docker-Image ist das [Docker Hub](https://hub.docker.com)):

```sh
docker run --rm -it ubuntu bash
```

> Der Parameter `--rm` entsorgt den Container nach Beendigung und die Parameter `-it` allokieren ein interkatives pseudo-Terminal (TTY).

### Liste aller (laufenden) Docker Container

Die Docker-App für Windows und Mac bietet ein "Dashboard" über das sich die laufenden Container inspizieren lassen. Daneben kann mit

```sh
docker ps
```

eine Liste der laufenden Container im Terminal ausgegeben werden:

```
$ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                NAMES
c342b9192b08        docker/getting-started   "nginx -g 'daemon of…"   8 hours ago         Up 8 hours          0.0.0.0:80->80/tcp   nostalgic_hertz
```

Die `CONTAINER ID` erlaubt den Container zu adressieren:

```sh
docker kill c342b9192b08         # Beendet den Container
docker exec -it c342b9192b08 sh  # Öffnet eine Shell (beachten `-it`-Parameter)
docker exec c342b9192b08 ls -lsa # Listet das Root-Verzeichnis (keine Shell)
```

**Merke:** Mit `docker exec` lässt sich etwas auf einem **laufenden*** Container ausführen.

### Dockerfile

Das Beispiel-Projekt ist eine Node.js Anwendung, daher starten wir mit einem `node` Image:

```Dockerfile
FROM node:12                      # Unser Basis-Image (siehe Docker Hub)
WORKDIR /app                      # Unser Arbeitsverzeichnis, wird angelegt, falls nicht vorhanden
COPY . .                          # Kopiert alle Dateien und Verzeichnisse aus dem aktuellen Verzeichnis in das Arbeitsverzeichnis des Containers
RUN yarn install                  # Führt `yarn install` im Container aus
CMD ["node", "/app/src/index.js"] # Definiert das Kommando, dass bei `docker run` ausgeführt wird
```

Das `Dockerfile` kommt in das Projektverzeichnis. Danach das Docker-Image mit

```sh
docker build -t getting-started .
```

Der Parameter `-t` vergibt einen Namen für das Image und `.` gibt den Pfad zum `Dockerfile` an (das aktuelle Verzeichnis). Nachdem das Image gebaut wurde, dann kann der Container gestartet werden:

```sh
docker run -dp 3000:3000 getting-started
```

Wobei der Parameter `-d` den Container im Hintergrund laufen lässt. Um den Container zu beenden muss er mit `docker kill CONTAINER_ID` beendet werden. Der Parameter `-p 3000:3000` macht den Port 3000 der Anwendung auf dem Host verfügbar. Nun kann die Beispielanwendung unter http://localhost:3000 aufgerufen werden.

### Persistenz der Daten

Wenn der Container beendet wird, gehen die Daten verloren. Das ist schlecht. Wir wollen die Daten unabhängig vom Container speicher. Antwort: **Volumes**:

```sh
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

Der Parameter `-v` erzeugt ein Volume mit dem Namen `todo-db` und hängt es in den Container unter dem Pfad `/etc/todos` ein. Alles was dort geschrieben wird, wird auf dem Volume gespeichert und geht **nicht** verloren wenn der Container beendet wird.

### Zugriff auf Daten vom Host

Um nicht nach jeder Code-Änderung das Image neu bauen zu müssen, können Verzeichnisse vom Host in einen Container "gemounted" werden. Der Parameter ist dabei der gleiche wie bei einem Volume:

```sh
docker run -dp 3000:3000 -v todo-db:/etc/todos -v ${PWD}:/app getting-started sh -c "yarn && yarn run dev"
```

Die relevanten Teile hier sind zum einen `-v ${PWD}:/app`, welches das aktuelle Host-Verzeichnis (`${PWD}`) in das Arbeitsverzeichnis (`/app`) im Container einhängt. Allerdings wird war bisher das `node_modules` Verzeichnis im Container vorhanden (der `docker build` Schritt hat das erzeugt) und in dem Verzeichnis auf dem Host ist es **nicht vorhanden**. Daher müssen die Abhängigkeiten erneut installiert werden. Allerdings muss dafür speziell einen `sh` gestartet werden. Zusätzlich muss nun die Anwendung mit `yarn run dev` gestartet werden, damit Änderungen am Code auch tatsächlich übernommen werden. Daher der sperrige Befehl `sh -c "yarn && yarn run dev"` am Ende.

### Mehrere Container via `docker-compose`

Um einen möglichst einfachen Befehl auf der Kommandozeile zu verwenden lässt sich die Konfiguration in ein `docker-compose.yml` File überführen:

```yaml
version: "3.7"
services:
  app:
    image: node:12
    command: sh -c "yarn install && yarn run dev"
    working_dir: /app
    ports:
      - 3000:3000
    volumes:
      - ./:/app
```

Nun verwenden wir nicht mal mehr unser eigenes `Dockerfile`, sondern installieren die Abhängigkeiten direkt via `command` in einen `node` Container.

Wenn wir nun zusätzlich noch eine Datenbank wie MySQL verwenden wollen, dann können wir das ebenfalls leicht der `docker-compose.yml` hinzufügen:

```yaml
version: "3.7"
services:
  app:
    image: node:12
    command: sh -c "yarn install && yarn run dev"
    working_dir: /app
    ports:
      - 3000:3000
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
  mysql:
    image: mysql:5.7
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos
    volumes:
      - todo-mysql-data:/var/lib/mysql

volumes:
  todo-mysql-data:
```

Am Ende lässt sich alles über ein einfaches

```sh
docker-compose up
```

starten und ebenso leicht über ein

```sh
docker-compose down
```

wieder beenden.

## Weitere Informationen

Detaillierte und weitere Informationen zu den einzelnen Aspekten finden sich im "Getting Started" von Docker selbst: https://github.com/docker/getting-started.
