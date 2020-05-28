# Web Development - Docker

Kurze Einführung in Docker basierend auf dem [offiziellen Docker "Getting Started"](https://github.com/docker/getting-started).

## "Getting Started"-App starten

Neben dem Beispiel-Code kann parallel die "Getting Started" App von Docker gestartet werden:

```
docker run --rm -dp 80:80 docker/getting-started
```

Im Anschluss lässt sich das Tutorial im Browser unter http://localhost aufrufen.

## Wichtige Befehle

Im folgenden wollen wir die wesentlichen Befehle zum Arbeiten mit Docker als Entwicklungsumgebung festhalten:

## Add MySQL container


```
docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
```
