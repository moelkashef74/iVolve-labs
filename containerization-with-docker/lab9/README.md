````md
# Lab 9: Containerized Node.js and MySQL Stack Using Docker Compose

## In this lab we are going to:

### 1- Create a Docker Compose configuration.

### 2- Deploy a Node.js application with a MySQL database.

### 3- Use Docker volumes for persistent database storage and application logs.

### 4- Push the application image to Docker Hub.

---



# Step 1: Create the Docker Compose File

Create a file named **compose.yml**.

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
       DB_HOST: db
       DB_USER: root
       DB_PASSWORD: 123
    volumes:
      - logs:/app/logs
    depends_on:
      - db

  db:
    image: mysql:5.7
    environment:
       MYSQL_ROOT_PASSWORD: 123
       MYSQL_DATABASE: ivolve
    volumes:
     - db_data:/var/lib/mysql
volumes:
  db_data:
  logs:


```

# Step 2: Build and Start the Containers

Build the application image and start both services.

```bash
docker compose up
```
```bash
db-1   | 2026-07-13T21:24:50.650145Z 0 [Note] mysqld: ready for connections.
db-1   | Version: '5.7.44'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
app-1  | ✅ Connected to MySQL and 'ivolve' DB found.
app-1  | 🚀 Server started on http://0.0.0.0:3000
```

Verify that the containers are running.

```bash
@moelkashef74 ➜ /workspaces/iVolve-labs (main) $ docker ps
```

Output:

```text
CONTAINER ID   IMAGE       COMMAND                  CREATED             STATUS          PORTS                                         NAMES
39f533b18b79   lab9-app    "docker-entrypoint.s…"   About an hour ago   Up 22 seconds   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   lab9-app-1
85c961f29433   mysql:5.7   "docker-entrypoint.s…"   About an hour ago   Up 23 seconds   3306/tcp, 33060/tcp                           lab9-db-1
```

# Step 3: Verify the Application

  test using curl:

```bash
curl http://localhost:3000
```

## Output:

```html
</main>

  <footer>
    &copy; 2025 iVolve Technologies. All rights reserved.
  </footer>
</body>
</html>
```

---

# Step 4: Verify the Application Logs

Generate requests to create access logs.

```bash
curl http://localhost:3000
curl http://localhost:3000/health
curl http://localhost:3000/ready
```

Display the access log inside the application container.

```bash
docker compose exec app cat /app/logs/access.log
```
or
```bash
cat /var/lib/docker/volumes/lab9logs/_data/access.log
```

## Output:

```text
GET / HTTP/1.1
GET /health HTTP/1.1
GET /ready HTTP/1.1
```
---

# Step 5: Tag the Docker Image

List available Docker images.

```bash
docker images
```

Tag the application image.

```bash
$ docker tag lab9-app:latest mosayed711/nodejs-app:v2
```

---

# Step 6: Login to Docker Hub

Login to Docker Hub. i used PAT (Personal Access Token) instead of password for security reasons.

```bash
docker login -u mosayed711
```

then add the token as the password.


# Step 7: Push the Image to Docker Hub

Push the image to Docker Hub.

```bash
docker push mosayed711/nodejs-app:v2
```

## Output:

```text
The push refers to repository [docker.io/mosayed711/nodejs-app]
v2: digest: sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx size:

```



# Step 8: Stop and Remove the Containers and Volumes

Stop the application stack.

```bash
docker compose down -v
```

---

