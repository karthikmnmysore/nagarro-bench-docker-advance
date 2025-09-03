# 1. Create a Volume
```
docker volume create my-volume
```
Check existing volumes:
```
docker volume ls
```
Inspect volume details:
```
docker volume inspect my-volume
```
# 🔹 2. Use the Volume with a Container
# Example: MySQL with Persistent Storage
Run MySQL with a named volume:
```
docker run -d --name my-mysql --network my-network -e MYSQL_ROOT_PASSWORD=rootpw -v my-volume:/var/lib/mysql mysql:8
```
-v my-volume:/var/lib/mysql → mounts the volume at MySQL’s data directory.
Now MySQL data persists even if the container is removed.

# 🔹 3. Test Persistence
Enter MySQL:
```
docker exec -it my-mysql mysql -uroot -prootpw
```
Create a database inside:
```
CREATE DATABASE testdb;
EXIT;
```
Remove the container:
```
docker rm -f my-mysql
```
Run MySQL again with the same volume:
```
docker run -d --name my-mysql --network my-network -e MYSQL_ROOT_PASSWORD=rootpw -v my-volume:/var/lib/mysql mysql:8
```
Connect again:
```
docker exec -it my-mysql mysql -uroot -prootpw
```
Check the databases:
```
SHOW DATABASES;
```
testdb → data persisted!

# 🔹 4. Manage Volumes
List volumes
```
docker volume ls
```
Clean up volume
```
docker volume prune
```
Remove a specific volume
```
docker volume rm my-volume
```

