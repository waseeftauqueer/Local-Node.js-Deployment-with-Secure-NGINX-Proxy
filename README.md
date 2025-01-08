# NGINX Proxy with Node.js and Docker

This project demonstrates how to set up a basic Express web server, containerize it using Docker, and proxy multiple instances of the app using NGINX. Additionally, the tutorial also covers how to secure your connection with SSL certificates.

## Prerequisites

- Node.js
- Docker
- NGINX
- SSL certificates (for secure connection)

## Table of Contents

- [Project Overview](#project-overview)
- [Setting Up the Express Web Server](#setting-up-the-express-web-server)
- [Containerizing the Application](#containerizing-the-application)
- [Creating Multiple Instances with Docker Compose](#creating-multiple-instances-with-docker-compose)
- [Setting Up NGINX as a Reverse Proxy](#setting-up-nginx-as-a-reverse-proxy)
- [Creating a Secure Encrypted Connection](#creating-a-secure-encrypted-connection)

## Project Overview

This project demonstrates:
1. A basic Express web server serving HTML files.
2. Dockerizing the application.
3. Running multiple instances of the app using Docker Compose.
4. Using NGINX as a reverse proxy to balance requests across multiple app instances.
5. Securing the connection using SSL certificates.

## Setting Up the Express Web Server

### Step 1: Create a Basic HTML File
Create a simple `index.html` file.

### Step 2: Create the Express Server

Create a `server.js` file with the following code:

```js
const express = require('express');
const path = require('path');
const app = express();
const port = 3000;

app.use('/images', express.static(path.join(__dirname, 'images')));

app.use('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
    console.log("Request served by node app");
});

app.listen(port, () => {
    console.log(`Node app is listening on port ${port}`);
});
```

### Step 3: Install Dependencies

Run the following command to install dependencies:

```bash
npm install
```

### Step 4: Run the Server

Run the server with:

```bash
node server.js
```

## Containerizing the Application

### Step 1: Create a Dockerfile

Create a `Dockerfile` with the following content:

```dockerfile
FROM node:14

WORKDIR /app

COPY server.js .
COPY index.html .
COPY images ./images
COPY package.json .

RUN npm install

EXPOSE 3000

CMD ["node", "server.js"]
```

### Step 2: Build and Run the Docker Image

Build the Docker image:

```bash
docker build -t myapp:1.0 .
```

Run the Docker container:

```bash
docker run -p 3000:3000 myapp
```

## Creating Multiple Instances with Docker Compose

### Step 1: Create `docker-compose.yaml`

Define multiple app instances in `docker-compose.yaml`:

```yaml
services:
  app1:
    build: .
    environment:
      - APP_NAME=App1
    ports:
      - "30001:3000"
  
  app2:
    build: .
    environment:
      - APP_NAME=App2
    ports:
      - "30002:3000"

  app3:
    build: .
    environment:
      - APP_NAME=App3
    ports:
      - "30003:3000"
```

### Step 2: Update the Server Code

Update the `server.js` to include the app name in the logs:

```js
const express = require('express');
const path = require('path');
const app = express();
const port = 3000;

const replicaApp = process.env.APP_NAME;

app.use('/images', express.static(path.join(__dirname, 'images')));

app.use('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
    console.log(`Request served by ${replicaApp}`);
});

app.listen(port, () => {
    console.log(`${replicaApp} is listening on port ${port}`);
});
```

### Step 3: Build and Run Multiple Instances

Build and start the instances:

```bash
docker-compose up --build -d
```

### Step 4: Check Logs

Check the logs to ensure the multiple instances are running correctly:

```bash
docker-compose logs
```

## Setting Up NGINX as a Reverse Proxy

### Step 1: Install NGINX

Install NGINX using:

```bash
sudo apt install nginx
```

### Step 2: Configure NGINX

Create an NGINX configuration file (`nginx.conf`) with the following content:

```yaml
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include mime.types;

    upstream nodejs_cluster {
        least_conn;
        server 127.0.0.1:30001;
        server 127.0.0.1:30002;
        server 127.0.0.1:30003;
    }

    server {
        listen 8080;
        server_name localhost;

        location / {
            proxy_pass http://nodejs_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real_IP $remote_addr;
        }
    }
}
```

### Step 3: Test the NGINX Proxy

Ensure NGINX is running and handling requests on port 8080.

## Creating a Secure Encrypted Connection

### Step 1: Generate SSL Certificates

Generate a self-signed SSL certificate using:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
```

### Step 2: Update NGINX for SSL

Update the NGINX configuration to use SSL:

```yaml
server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /path/to/nginx-selfsigned.crt;
    ssl_certificate_key /path/to/nginx-selfsigned.key;

    location / {
        proxy_pass http://nodejs_cluster;
    }
}
```

### Step 3: Restart NGINX

Restart NGINX to apply the changes.

---

## Conclusion

With this setup, you have:
- A basic Express web server.
- Dockerized the application and run multiple instances.
- Used NGINX as a reverse proxy for load balancing.
- Secured the connection using SSL certificates.

---
