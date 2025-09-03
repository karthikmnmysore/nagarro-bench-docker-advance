# Dockerfile
```
FROM openjdk:17-jdk-slim

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
COPY app.jar /app/app.jar
RUN chown -R appuser:appgroup /app

USER appuser
ENTRYPOINT ["java", "-jar", "app.jar"]
```
# docker-compose.yml
```
version: "3.3"

services:
  spring-app:
    build: .
    container_name: spring-app
    depends_on:
      - mysql-db
      - promtail
    ports:
      - "8080:8080" # optional, if your app has web endpoints
    logging:
      driver: "json-file" # default driver
      options:
        max-size: "10m"
        max-file: "3"

  mysql-db:
    image: mysql:8
    container_name: mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: rootpw
      MYSQL_DATABASE: appdb
      MYSQL_USER: appuser
      MYSQL_PASSWORD: appuserpw
    volumes:
      - db_data:/var/lib/mysql

  loki:
    image: grafana/loki:2.9.4
    container_name: loki
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:2.9.4
    container_name: promtail
    volumes:
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml
    depends_on:
      - loki

  grafana:
    image: grafana/grafana:10.4.2
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - loki

volumes:
  db_data:
```
# Build and run
```
docker-compose up -d --build
```
# promtail-config.yml
```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker
          __path__: /var/lib/docker/containers/*/*.log
```

# View logs in Grafana
Open Grafana → http://localhost:3000

Default login: admin / admin

Add Loki as data source:

URL: http://loki:3100

Go to Explore → Loki

Run query:
```
{job="docker"}
```
Output:
```
Hello Spring Boot
Hello Spring Boot
Hello Spring Boot
```

# How it works
**spring-app** writes logs to the container’s JSON log file (/var/lib/docker/containers/.../*.log).

**Promtail** tails these logs and ships them to **Loki**.

**Grafana** connects to **Loki** as a data source.

We can visualize logs in Grafana (http://localhost:3000, login: admin/admin).
