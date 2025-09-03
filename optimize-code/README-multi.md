## 🏗 Multi-Stage Build

We also provide a multi-stage Dockerfile (`Dockerfile.multi`) that further reduces image size:

```bash
docker build -f Dockerfile.multi -t node-app-multi .
docker images | grep node-app


# Project Structure
```
├── Dockerfile.multi
├── package.json      # Node.js dependencies
├── server.js         # Minimal Express app
└── .dockerignore     # Files excluded from build
```

# Build the Multi-stage image
```
docker build -f Dockerfile.multi -t node-app-multi .
```

```

# Compare Image Sizes
```
docker images | grep node-app
node-app-multi       latest      8d339ae2ef35   About a minute ago   194MB
node-app-optimized   latest      b0a1108fc603   About an hour ago    199MB
node-app-old         latest      4d2465af4b10   About an hour ago    1.1GB
```

# Run the Container
```
docker run --rm -p 3000:3000 node-app-multi
```
Open In browser at http://localhost:3000
 → you should see
 ```
Hello from Docker!
```
