```
karthik@IN-PW0EYYJQ:/mnt/c/Users/karthikn/karthik-bench$ cat Dockerfile
FROM nginx:alpine

# Remove "user ..." and replace "pid" path
RUN sed -i '/^user /d' /etc/nginx/nginx.conf \
    && sed -i 's|^pid .*|pid /home/appuser/nginx.pid;|' /etc/nginx/nginx.conf

# Create non-root user and group
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Create necessary directories and fix permissions
RUN mkdir -p /var/cache/nginx /var/log/nginx /home/appuser \
    && chown -R appuser:appgroup /var/cache/nginx /var/log/nginx /home/appuser /etc/nginx

# Switch to non-root user
USER appuser

# Expose application port
EXPOSE 8080

# Add a healthcheck
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -q --spider http://localhost:8080/ || exit 1

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
export DOCKER_CONTENT_TRUST=0
```

# Build and run
```
docker build -t secure-nginx .
docker run -d --name secure-nginx -p 8081:8080 secure-nginx
```
# Check Docker Bench:
```
docker run -it --net host --pid host --userns host \
   -v /var/run/docker.sock:/var/run/docker.sock:ro \
   -v /etc:/etc:ro -v /usr/bin/docker-containerd:/usr/bin/docker-containerd:ro \
   docker/docker-bench-security
```

# Testing
```
[INFO] 4 - Container Images and Build File
[PASS] 4.1  - Ensure a user for the container has been created
[NOTE] 4.2  - Ensure that containers use trusted base images
[NOTE] 4.3  - Ensure unnecessary packages are not installed in the container
[NOTE] 4.4  - Ensure images are scanned and rebuilt to include security patches
[WARN] 4.5  - Ensure Content trust for Docker is Enabled
[WARN] 4.6  - Ensure HEALTHCHECK instructions have been added to the container image
```
Check in “4” section, it is “PASSED”
Non Root user is added then Health check is added in Dockerfile, so Docker check is passed

