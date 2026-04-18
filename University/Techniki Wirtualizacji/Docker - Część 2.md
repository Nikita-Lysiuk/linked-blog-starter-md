Aby utworzyć obraz, napisałem plik Dockerfile, w którym opisałem, że tworzę nowy obraz na bazie Ubuntu, eksponuję na porcie 8085, wskazuję wolumin /data i wskazuję punkt wejścia naszego własnego skryptu entrypoint.sh, który tworzy certyfikaty, jeśli nie ma ich w naszym woluminie, a jeśli są, to wykorzystuje istniejące. Tworzy certyfikaty za pomocą openssl i podpisuje za pomocą rsa z modułem 2048. Następnie skrypt łączy nasz volume z folderem, w którym cockpit będzie szukał certyfikatów podczas uruchamiania, a mianowicie: /etc/cockpit/ws-certs.d, i uruchamia sam cockpit-ws na porcie 8085.

```Dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
    cockpit-ws \
    openssl && \
    rm -rf /var/lib/apt/lists/*

  

COPY entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh

EXPOSE 8085/tcp

VOLUME ["/data"]

ENTRYPOINT ["/entrypoint.sh"]
```

```sh
#!/bin/bash

set -e

CERT_KEY="/data/cockpit.key"
CERT_CRT="/data/cockpit.crt"

if [ ! -f "$CERT_KEY" ] || [ ! -f "$CERT_CRT" ]; then

  echo ">>> Generating self-signed certificate in /data..."
  openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout "$CERT_KEY" \
    -out "$CERT_CRT" \
    -subj "/CN=localhost"
else
  echo ">>> Using existing certificate from /data/"
fi

CERT_DIR="/etc/cockpit/ws-certs.d"

mkdir -p "$CERT_DIR"


ln -sf "$CERT_KEY" "$CERT_DIR/docker-volume.key"
ln -sf "$CERT_CRT" "$CERT_DIR/docker-volume.cert"

echo ">>> Starting Cockpit"

exec /usr/lib/cockpit/cockpit-ws --port 8085
```

Za pomocą następujących poleceń utworzyłem wolumin, zebrałem obraz i uruchomiłem kontener, łącząc wolumin, nadałem nazwę i uruchomiłem w tle. 
`docker volume create cockpit_data`
`docker build -t mycockpit-image .`
`docker run -d --name mycockpit -p 8085:8085 -v cockpit_data:/data mycockpit-image`
`docker logs mycockpit`
``` sh
❯ docker logs mycockpit
>>> Generating self-signed certificate in /data...

>>> Starting Cockpit
cockpit-protocol-Message: 19:48:40.995: couldn't read from connection: Peer sent fatal TLS alert: Unknown certificate
```
Z logów wynika, że połączenie się powiodło, ale nie udało się zweryfikować ważności certyfikatów, ponieważ są one samopodpisane.


```  
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    container_name: gitlab
    restart: unless-stopped
    hostname: 'gitlab.nikita_lysiuk.com'
    ports:
      - "80:80"
      - "443:443"
      - "2222:22"
    volumes:
      - gitlab_data:/var/opt/gitlab
    networks:
      - gitlab-network
    depends_on:
      - postgres
      - redis
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://192.168.0.127'

        postgresql['enable'] = false
        gitlab_rails['db_adapter'] = 'postgresql'
        gitlab_rails['db_encoding'] = 'unicode'
        gitlab_rails['db_database'] = 'database_gitlab'
        gitlab_rails['db_username'] = 'gitlab'
        gitlab_rails['db_password'] = 'password'
        gitlab_rails['db_host'] = 'postgres'
  
        redis['enable'] = false
        gitlab_rails['redis_host'] = 'redis'
        gitlab_rails['redis_port'] = 6379

        gitlab_rails['ssh_port'] = 2222

  postgres:
    image: 'postgres:latest'
    container_name: gitlab-postgres
    restart: unless-stopped
    volumes:
      - pg_data:/var/lib/postgresql
    networks:
      - gitlab-network
    environment:
      POSTGRES_DB: database_gitlab
      POSTGRES_USER: gitlab
      POSTGRES_PASSWORD: password
  redis:
    image: 'redis:7-alpine'
    container_name: gitlab-redis
    restart: unless-stopped
    volumes:
      - redis_data:/data
    networks:
      - gitlab-network

volumes:
  pg_data:
  redis_data:
  gitlab_data:
  
networks:
  gitlab-network:
    driver: bridge
```


![[Pasted image 20251111222204.png]]Napisałem plik docker.compose, który spełnia wszystkie wymagania, a także przetestowałem go, otwierając gitlab i logując się jako administrator za pomocą root i hasła, które uzyskałem za pomocą `docker exec -it gitlab grep "Password:" /etc/gitlab/initial_root_password.`