# Lab 06 - Managing Docker Environment Variables Across Build and Runtime



## Step 1: Clone the Application Source Code

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-3.git
```

---

## Step 2: Create the Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . .

RUN pip install flask

ENV APP_MODE=production
ENV APP_REGION=canada-west

EXPOSE 8080

CMD ["python", "app.py"]
```

---

## Step 3: Build the Docker Image

```bash
docker build -t flask-app:v1 .
```

Expected output:

```text
Successfully built <IMAGE_ID>
Successfully tagged flask-app:v1
```

---

## Step 4: Method 1 - Pass Environment Variables in the `docker run` Command

Run the container:

```bash
docker run -d \
--name flask-dev \
-p 8081:8080 \
-e APP_MODE=development \
-e APP_REGION=us-east \
flask-app:v1
```


Expected output:

```text
APP_MODE=development
APP_REGION=us-east
```

---

## Step 5: Method 2 - Pass Environment Variables Using an Environment File

Create an environment file named `app.env`:

```text
APP_MODE=staging
APP_REGION=us-west
```

Run the container:

```bash
docker run -d \
--name flask-staging \
-p 8082:8080 \
--env-file app.env \
flask-app:v1
```



## Step 6: Method 3 - Define Environment Variables in the Dockerfile

The Dockerfile already contains:

```dockerfile
ENV APP_MODE=production
ENV APP_REGION=canada-west
```

Run the container:

```bash
docker run -d \
--name flask-prod \
-p 8083:8080 \
flask-app:v1
```



## Step 7: Verify Running Containers

```bash
docker ps
```

Expected output:

```text
CONTAINER ID   IMAGE          STATUS         PORTS                    NAMES
<CONTAINER_ID> flask-app:v1   Up             0.0.0.0:8081->8080/tcp   flask-dev
<CONTAINER_ID> flask-app:v1   Up             0.0.0.0:8082->8080/tcp   flask-staging
<CONTAINER_ID> flask-app:v1   Up             0.0.0.0:8083->8080/tcp   flask-prod
```

---

## Step 8: Stop the Containers

```bash
docker stop flask-dev flask-staging flask-prod
```

Expected output:

```text
flask-dev
flask-staging
flask-prod
```

---

## Step 9: Remove the Containers

```bash
docker rm flask-dev flask-staging flask-prod
```

