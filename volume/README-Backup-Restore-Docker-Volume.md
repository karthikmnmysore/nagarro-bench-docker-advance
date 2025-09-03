# 🔹 Backup & Restore Docker Volumes
# Step 1: Create a Test Volume and Container
Example with MySQL (using a named volume db_data):
```
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=rootpw -v db_data:/var/lib/mysql mysql:8
```
# Step 2: Backup a Volume
We’ll use a throwaway container to tar the volume contents.
```
docker run --rm -v db_data:/volume -v $(pwd):/backup alpine tar czf /backup/db_data_backup.tar.gz -C /volume .
```
# Step 3: Remove the Volume (simulate data loss)
```
docker rm -f mysql-db
docker volume rm db_data
```
# Step 4: Restore the Volume
Create an empty volume:
```
docker volume create db_data
```
Restore from the backup:
```
docker run --rm -v db_data:/volume -v $(pwd):/backup alpine tar xzf /backup/db_data_backup.tar.gz -C /volume
```
# Step 5: Verify Restoration
```
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=rootpw -v db_data:/var/lib/mysql mysql:8
```
**Backup format** → we used .tar.gz, but you can use plain .tar.

**Automation** → can be added to a cronjob or CI/CD for regular backups.

**Portability** → backup file can be copied to another server and restored there.


# What below command means
```
docker run --rm -v db_data:/volume -v $(pwd):/backup alpine tar czf /backup/db_data_backup.tar.gz -C /volume .
```
```
docker run --rm
```
Runs a temporary container.

**--rm** means Docker will automatically remove the container once it exits (no leftovers).
```
-v db_data:/volume
```
Mounts the Docker named volume db_data (where MySQL stores data) inside the container at /volume.
So, inside this container, /volume = MySQL’s data directory.
```
-v $(pwd):/backup
```
Mounts your current host directory ($(pwd)) into the container at /backup.
This is where the backup file will be written.
After command runs, the backup tarball appears in your host directory.

alpine
The container uses the lightweight Alpine Linux image.
It’s tiny but has the tar utility we need.
```
tar czf /backup/db_data_backup.tar.gz -C /volume .
```
Runs inside the container.

# What actually happens
Alpine container starts.

It sees MySQL volume mounted at /volume.

It sees host directory mounted at /backup.

It creates a gzipped tarball of /volume and saves it as /backup/db_data_backup.tar.gz.

Since /backup is mounted to your host, you now have the backup file in your current directory.

Container exits and is auto-removed (--rm).
