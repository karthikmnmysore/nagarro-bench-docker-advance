# LAMP-like stack:
**MySQL** → stores data in a named volume (persistent DB).
**phpMyAdmin** → web UI for MySQL (connected via network).
**Nginx** → serves static files from a bind mount on the host.

docker-compose.yml
```
version: "3.3"

services:
  db:
    image: mysql:8
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpw
      MYSQL_DATABASE: myappdb
    volumes:
      - db_data:/var/lib/mysql   # Named volume for persistence
    networks:
      - app-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    restart: always
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: rootpw
    ports:
      - "8080:80"                # Access phpMyAdmin at http://localhost:8080
    networks:
      - app-network

  nginx:
    image: nginx:latest
    container_name: nginx-web
    ports:
      - "8081:80"                # Access Nginx at http://localhost:8081
    volumes:
      - ./html:/usr/share/nginx/html:ro   # Bind mount (host ./html folder → container)
    networks:
      - app-network

# Named volume
volumes:
  db_data:

# Shared network
networks:
  app-network:

```
# 🔹 How It Works
# Named Volume (db_data)
MySQL stores data in /var/lib/mysql.
Data persists even if the container is deleted.
# Bind Mount (./html)
Your host directory ./html is mounted inside Nginx.
Edit ./html/index.html → refresh browser → changes show instantly.
# Network (app-network)
All services can talk to each other (e.g., phpMyAdmin → MySQL).

# 🔹 Steps to Test
Create a local folder for Nginx static files:
```
mkdir html
echo "<h1>Hello from Nginx + Bind Mount</h1>" > html/index.html
```
Run the stack:
```
docker-compose up -d
```
# Open in browser:
http://localhost:8080 → phpMyAdmin (connects to MySQL using named volume).
http://localhost:8081 → Nginx serving your host html/index.html.

# Test persistence:
Create a DB in phpMyAdmin.
Stop containers: docker-compose down.
Start again: docker-compose up -d.
Database is still there (thanks to named volume).

# Test bind mount:
Edit html/index.html on your host:
```
<h1>Updated directly from host!</h1>
```
Refresh http://localhost:8081 → change is reflected immediately.

👉 This way, you clearly see:
**Named volumes** → persistent, managed by Docker (good for DBs).
**Bind mounts** → live sync with host files (good for dev, configs, static content).
