# Smaller base image
FROM node:18-slim

WORKDIR /app

# Copy package.json
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy app source last
COPY . .

CMD ["node", "server.js"]
