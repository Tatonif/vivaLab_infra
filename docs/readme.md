so ging es loss :
Git Repository erstellen:

Erstelle ein neues Repository auf GitHub oder GitLab. Nenne es z.B. vivaLab_infrastructure.
Klone das Repository auf deinen lokalen Rechner.
sh
Copy code
git clone https://github.com/dein-benutzername/vivaLab_infrastructure.git
cd vivaLab_infrastructure
Verzeichnisstruktur anlegen:

Erstelle die folgende Verzeichnisstruktur:
sh
Copy code
mkdir -p apps/{wordpress,jira,mediawiki,linux_custom}
mkdir monitoring
mkdir docs
mkdir scripts
2. Docker-Konfigurationen erstellen
WordPress:

Erstelle eine docker-compose.yml Datei im apps/wordpress Verzeichnis:
yml
Copy code
version: '3.8'
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
  db:
    image: mysql:5.7
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: rootpassword
volumes:
  db_data:
JIRA:

Erstelle eine docker-compose.yml Datei im apps/jira Verzeichnis:
yml
Copy code
version: '3.8'
services:
  jira:
    image: atlassian/jira-software:latest
    ports:
      - "8081:8080"
    environment:
      ATL_JDBC_URL: jdbc:postgresql://db:5432/jiradb
      ATL_JDBC_USER: jira
      ATL_JDBC_PASSWORD: jirapassword
  db:
    image: postgres:latest
    environment:
      POSTGRES_DB: jiradb
      POSTGRES_USER: jira
      POSTGRES_PASSWORD: jirapassword
volumes:
  db_data:
MediaWiki:

Erstelle eine docker-compose.yml Datei im apps/mediawiki Verzeichnis:
yml
Copy code
version: '3.8'
services:
  mediawiki:
    image: mediawiki:latest
    ports:
      - "8082:80"
    environment:
      MEDIAWIKI_DB_HOST: db
      MEDIAWIKI_DB_USER: mediawiki
      MEDIAWIKI_DB_PASSWORD: mediawikipassword
      MEDIAWIKI_DB_NAME: mediawiki
  db:
    image: mariadb:latest
    environment:
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: mediawiki
      MYSQL_PASSWORD: mediawikipassword
      MYSQL_ROOT_PASSWORD: rootpassword
volumes:
  db_data:
Custom Linux Image mit NMAP und SSH:

Erstelle eine Dockerfile Datei im apps/linux_custom Verzeichnis:

Dockerfile
Copy code
FROM alpine:latest

RUN apk update && \
    apk add --no-cache nmap openssh && \
    echo "root:password" | chpasswd && \
    sed -i 's/#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    mkdir /root/.ssh && \
    touch /root/.ssh/authorized_keys

EXPOSE 22
CMD ["sh", "-c", "rc-service sshd start && tail -f /dev/null"]
Erstelle eine docker-compose.yml Datei im apps/linux_custom Verzeichnis:

yml
Copy code
version: '3.8'
services:
  linux_custom:
    build: .
    ports:
      - "2222:22"
3. Monitoring einrichten
Portainer:

Erstelle eine docker-compose.yml Datei im monitoring Verzeichnis:
yml
Copy code
version: '3.8'
services:
  portainer:
    image: portainer/portainer-ce:latest
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
volumes:
  portainer_data:
Prometheus und Alertmanager:

Erstelle eine docker-compose.yml Datei im monitoring Verzeichnis:

yml
Copy code
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
Erstelle die prometheus.yml Datei:

yml
Copy code
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
Erstelle die alertmanager.yml Datei:

yml
Copy code
global:
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'user'
  smtp_auth_password: 'password'

route:
  receiver: 'email'

receivers:
  - name: 'email'
    email_configs:
      - to: 'admin@example.com'
