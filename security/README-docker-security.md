# Generate app.jar file from Windows laptop
```
cat <<'EOF' > Dummy.java
public class Dummy {
    public static void main(String[] args) throws Exception {
        System.out.println("Hello Spring Boot");
        Thread.sleep(Long.MAX_VALUE); // keeps app alive
    }
}
EOF
javac Dummy.java
echo "Main-Class: Dummy" > manifest.txt
jar cfm app.jar manifest.txt Dummy.class
```

# Create Dockerfile
Dockerfile
```
# Stage 1: Build
FROM openjdk:17-jdk-slim AS build

WORKDIR /app
COPY app.jar /app/app.jar

# Stage 2: Run as non-root
FROM openjdk:17-jdk-slim

# Create a non-root user and group
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
COPY --from=build /app/app.jar /app/app.jar

# Change ownership to non-root user
RUN chown -R appuser:appgroup /app

# Switch to non-root
USER appuser

ENTRYPOINT ["java", "-jar", "app.jar"]
```

# docker-compose.yml
```
version: "3.3"

services:
  mysql:
    image: mysql:8
    container_name: mysql-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - db_data:/var/lib/mysql
    secrets:
      - db_password
    networks:
      - app_net

  app:
    build: .
    container_name: spring-app
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-db:3306/appdb
      SPRING_DATASOURCE_USERNAME: appuser
      SPRING_DATASOURCE_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    networks:
      - app_net
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true

volumes:
  db_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt

networks:
  app_net:
    driver: bridge
```
# Secret management demo
```
echo "supersecretpassword" > db_password.txt
```
# Build and start
```
docker-compose up -d --build
```

# Scan the image for vulnerabilities
Scan using CICD
```
trivy image secure-springboot-app
```

# Least privilege runtime
```
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
```
no-new-privileges → prevents privilege escalation.

cap_drop: ALL → removes all Linux kernel capabilities.

read_only: true → makes the container’s filesystem read-only (except mounted volumes/secrets).

# Verify
Check which user is running inside the container:
```
docker exec -it spring-app whoami
```
You should see:
```
appuser
```
If you try to access /root now:
```
docker exec -it spring-app ls /root
```
You’ll get **permission denied**

# Why you still “see” root files
Every container is built from a Linux image (debian, ubuntu, alpine, etc.), which contains system directories.

They belong to **root inside the image**, but Docker does not strip them away — otherwise, your app wouldn’t even run.

What changed is permissions:

You can ls /etc because it’s world-readable.

You cannot ls /root or edit /etc/shadow, because appuser has no rights.

If you try touch /etc/hosts, you’ll see **Permission denied**.

# Security perspective
Running as appuser ensures if someone breaks into your container, they cannot **escalate privileges** (without an exploit).

They can’t affect the Docker host (/root on host is completely separate).

They’re jailed inside the container’s filesystem.

# With above steps we now have:
```
Non-root container

Least privilege runtime

Secret mounted securely

Vulnerability scanning support
```
