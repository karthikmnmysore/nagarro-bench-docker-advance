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
