# 🐳 Docker Project 1 - Containerized Nginx Website

## 📌 Project Overview

This is my first Docker project.

I created a simple HTML website and deployed it inside an Nginx Docker container running on an Ubuntu EC2 instance.

The project helped me understand the complete Docker workflow:

**HTML Application → Dockerfile → Docker Image → Docker Container → Nginx → Browser**

---

## 🏗️ Architecture

```text
                  Internet
                     |
                     v
              EC2 Public IP
                     |
                     | Port 8080
                     v
             EC2 Security Group
                     |
                     v
             Docker Host (EC2)
                     |
                     | 8080:80
                     v
          +----------------------+
          |   Nginx Container    |
          |                      |
          |   Port 80            |
          |                      |
          |   index.html         |
          +----------------------+
```

## 🛠️ Technologies Used

- Ubuntu Linux
- AWS EC2
- Docker
- Nginx
- HTML
- Docker Networking
- Docker Volumes
- Docker Bind Mounts

## 📁 Project Structure

```text
docker-project1/
│
├── Dockerfile
├── index.html
└── README.md
```

## 🐳 Docker Installation

Docker was installed on Ubuntu using:

```bash
sudo apt update
sudo apt install docker.io -y
```

Start Docker:

```bash
sudo systemctl start docker
```

Enable Docker at system startup:

```bash
sudo systemctl enable docker
```

Check Docker version:

```bash
sudo docker --version
```

Test Docker:

```bash
sudo docker run hello-world
```

## 👤 Run Docker Without sudo

Added the current Linux user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

After reconnecting to the EC2 instance:

```bash
docker ps
```

Docker can then be used without `sudo`.

## 🌐 Creating the Website

Created an HTML file:

```bash
nano index.html
```

Example:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Project</title>
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>My first website is running inside a Docker container.</p>
</body>
</html>
```

## 📄 Dockerfile

The Dockerfile used:

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html
```

### Explanation

#### FROM

```dockerfile
FROM nginx:latest
```

Uses the Nginx image as the base image.

#### COPY

```dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

Syntax:

```text
COPY SOURCE DESTINATION
```

Source:

```text
index.html
```

Destination:

```text
/usr/share/nginx/html/index.html
```

The HTML file is copied into the Nginx web root inside the Docker image.

## 🏗️ Building the Docker Image

```bash
docker build -t my-docker-website .
```

### Explanation

```text
docker build
    → Builds a Docker image

-t my-docker-website
    → Assigns the image name

.
    → Uses the current directory as the build context
```

Check images:

```bash
docker images
```

## 🚀 Running the Container

```bash
docker run -d -p 8080:80 --name my-website my-docker-website
```

### Explanation

`-d` runs the container in detached/background mode.

`-p` maps a host port to a container port:

```text
-p HOST_PORT:CONTAINER_PORT
```

In this project:

```text
8080:80
```

means:

```text
EC2 Port 8080
       |
       v
Container Port 80
       |
       v
Nginx
```

`--name my-website` gives the container a readable name.

## ☁️ AWS EC2 Security Group

Port 8080 must be allowed in the EC2 Security Group.

Example:

```text
Type: Custom TCP
Port: 8080
```

For learning/testing, the port was temporarily opened for external access.

For production, access should be restricted to trusted IP addresses or placed behind a reverse proxy/load balancer.

## 🔍 Docker Container Commands

Show running containers:

```bash
docker ps
```

Show all containers:

```bash
docker ps -a
```

Stop a container:

```bash
docker stop my-website
```

Start a stopped container:

```bash
docker start my-website
```

Remove a container:

```bash
docker rm my-website
```

View container logs:

```bash
docker logs my-website
```

Inspect a container:

```bash
docker inspect my-website
```

## 🔄 Image vs Container

A Docker image is a packaged template.

A container is a running instance of that image.

```text
Dockerfile
    |
    | docker build
    v
Docker Image
    |
    | docker run
    v
Docker Container
```

Multiple containers can be created from the same image.

## 🔄 Updating the Application

If the HTML file is changed after the image is built, the existing container does not automatically receive the change.

The normal workflow is:

```text
Modify application
       |
       v
docker build
       |
       v
New Image
       |
       v
New Container
```

Example:

```bash
docker build -t my-docker-website .
```

Then recreate the container:

```bash
docker stop my-website
docker rm my-website
docker run -d -p 8080:80 --name my-website my-docker-website
```

## 📂 Bind Mount

A bind mount connects a directory on the EC2 host directly to a directory inside the container.

Example:

```bash
docker run -d \
  -p 8080:80 \
  --name my-website \
  -v $(pwd):/usr/share/nginx/html \
  nginx:latest
```

Syntax:

```text
-v HOST_PATH:CONTAINER_PATH
```

With a bind mount, changes made to `index.html` on the EC2 can immediately appear inside the container.

This is useful during development.

## 💾 Docker Volumes

Create a Docker-managed volume:

```bash
docker volume create my-website-data
```

List volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect my-website-data
```

Use the volume:

```bash
docker run -d \
  -p 8080:80 \
  --name my-website \
  -v my-website-data:/usr/share/nginx/html \
  nginx:latest
```

Volume syntax:

```text
-v VOLUME_NAME:CONTAINER_PATH
```

Docker manages the storage location of the volume.

### Why Volumes Are Important

Container data can disappear when a container is removed.

Volumes allow data to survive independently of the container.

```text
Container
    |
    v
Docker Volume
    |
    v
Persistent Data
```

Volumes are especially important for databases such as MySQL, PostgreSQL, MongoDB, and Redis.

## 🌐 Docker Networking

Create a custom Docker network:

```bash
docker network create my-app-network
```

List networks:

```bash
docker network ls
```

Inspect a network:

```bash
docker network inspect my-app-network
```

Run a container on the network:

```bash
docker run -d \
  --name my-website \
  --network my-app-network \
  -p 8080:80 \
  nginx:latest
```

## 🔗 Container-to-Container Communication

Create another container:

```bash
docker run -d \
  --name client \
  --network my-app-network \
  alpine \
  sleep 3600
```

Enter the container:

```bash
docker exec -it client sh
```

Test communication with the Nginx container:

```bash
wget -qO- http://my-website
```

Docker's internal DNS allows containers on the same custom network to communicate using container names.

```text
client
   |
   | HTTP request
   v
my-website
   |
   v
Nginx
```

No hard-coded container IP address was required.

## 📚 Important Docker Commands

| Command | Purpose |
|---|---|
| `docker --version` | Check Docker version |
| `docker images` | List images |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker build` | Build an image |
| `docker run` | Create and start a container |
| `docker start` | Start stopped container |
| `docker stop` | Stop running container |
| `docker rm` | Remove container |
| `docker logs` | View container logs |
| `docker exec` | Execute command inside container |
| `docker inspect` | View detailed information |
| `docker volume create` | Create a volume |
| `docker volume ls` | List volumes |
| `docker volume inspect` | Inspect a volume |
| `docker network create` | Create network |
| `docker network ls` | List networks |
| `docker network inspect` | Inspect network |

## 🧠 Key Concepts Learned

1. **Dockerfile** — Instructions used to create a Docker image.
2. **Image** — A packaged template used to create containers.
3. **Container** — A running instance of an image.
4. **Port Mapping** — `HOST:CONTAINER`, for example `8080:80`.
5. **Bind Mount** — Connects a host directory directly to a container directory.
6. **Docker Volume** — Docker-managed persistent storage.
7. **Docker Network** — Allows containers to communicate with each other.
8. **Docker DNS** — Containers on a custom network can communicate using container names.

## 🎯 Project Outcome

Successfully deployed an Nginx website inside a Docker container running on an AWS EC2 Ubuntu server.

The project covered fundamental Docker concepts required before moving into multi-container applications and Docker Compose.

## 🚀 Next Steps

The next project will build a real multi-container application using:

```text
Flask
   +
Redis
   +
Docker Network
```

After that:

```text
Docker Compose
   ↓
Frontend
   ↓
Backend
   ↓
Database
```
