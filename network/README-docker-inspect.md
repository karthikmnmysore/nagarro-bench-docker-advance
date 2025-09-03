# 1. Create a Custom Network
```
docker network create my-network
docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
e57f55043f8b   my-network           bridge    local
```
# 2. Run Multiple Containers in the Same Network
# Example: Nginx + BusyBox (ping test)
Run Nginx in the custom network:
```
docker run -d --name my-nginx --network my-network nginx
```
Run a BusyBox container in the same network:
```
docker run -it --rm --name my-busybox --network my-network busybox sh
```
Inside BusyBox shell, try:
```
ping my-nginx
```
Exit BusyBox:
```
exit
```
# Step 3: Inspect the Network
```
docker network inspect my-network
```
# Step 4: Manage the Network
# Connect another container to it:
```
docker network connect my-network my-nginx
```
Disconnect a container:
```
docker network disconnect my-network my-nginx
```
Remove network (after containers are stopped/disconnected):
```
docker network rm my-network
```
Clean up unused networks:
```
docker network prune
```
