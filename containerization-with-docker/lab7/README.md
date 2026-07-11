# Lab 7: Docker Volumes and Bind Mounts with Nginx

## in this lab we are going to use volumes in docker with the 2 ways :
### 1- creating docker volume, its fully managedby docker 
### 2- using Bind Mounts 

## Step 1: Create a Docker Volume

```bash
docker volume create nginx_logs
```


verify it in the default volumes path.

```bash
root ➜ .../codespaces-blank/iVolve-labs/containerization-with-docker/lab7 (main) $ ls /var/lib/docker/volumes/
backingFsBlockDev  metadata.db        nginx_logs/
```
---

## Step 2: Create index.html file with “Hello from Bind Mount”

```bash
echo "Hello from Bind Mount" > index.html
```

---

## Step 3: Run Nginx container with the 2 mount types 

```bash
docker run -d \
  --name nginx \
  -p 8080:80 \
  -v nginx_logs:/var/log/nginx \
  -v ./index.html:/usr/share/nginx/html/index.html \
  nginx
```

---

## Step 4: Verify Nginx page by running


```bash
root ➜ .../codespaces-blank/iVolve-labs/containerization-with-docker/lab7 (main) $ curl loacalhost:8080
Hello from Bind Mount
```

---

## Step 5: Change in the index.html file in your local machine then verify Nginx page again



```bash
echo "\n this page has been modified" >> index.html 
curl loacalhost:8080

```

output:

```text
Hello from Bind Mount
 this page has been modified
```

---



## Step 6: Verify Logs Are Stored in the Volume

Generate some requests.

```bash
curl http://localhost:8080
curl http://localhost:8080
```

Inspect the volume.

```bash
docker volume inspect nginx_logs
```

Example output:

```text
Mountpoint:
/var/lib/docker/volumes/nginx_logs/_data
```

View the log files.

```bash
sudo ls /var/lib/docker/volumes/nginx_logs/_data
```

You should see files similar to:

```text
access.log
error.log
```


---

## Step 7: Remove the Container

Stop and remove the container.

```bash
docker stop nginx
docker rm nginx
```

---

## Step 8: Delete the Volume

Remove the Docker volume.

```bash
docker volume rm nginx_logs
```

Verify it has been removed.

```bash
docker volume ls
```

---

