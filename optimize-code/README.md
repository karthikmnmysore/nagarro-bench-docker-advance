# Node.js Docker Image Optimization Demo
This project demonstrates how to optimize a Dockerfile for faster builds and smaller image size.
We provide two versions of the Dockerfile:
  Dockerfile.old → Unoptimized version
  Dockerfile → Optimized version
The application itself is a minimal Node.js app (server.js) using Express.

# Project Structure
```
├── Dockerfile        # Optimized Dockerfile
├── Dockerfile.old    # Unoptimized Dockerfile
├── package.json      # Node.js dependencies
├── server.js         # Minimal Express app
└── .dockerignore     # Files excluded from build
```

# Build the Images
# 1. Build the unoptimized image
```
docker build -f Dockerfile.old -t node-app-old .
```
# 2. Build the optimized image
```
docker build -f Dockerfile -t node-app-optimized .
```

# Compare Image Sizes
```
docker images | grep node-app
node-app-optimized   latest      b0a1108fc603   5 seconds ago   199MB
node-app-old         latest      4d2465af4b10   4 minutes ago   1.1GB
```

# Run the Container
```
docker run --rm -p 3000:3000 node-app-optimized
```
Open In browser at http://localhost:3000
 → you should see
 ```
Hello from Docker!
```
