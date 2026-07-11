# Lab 8: Custom Docker Network for Microservices

## In this lab we are going to :
### 1- Building custom Docker images for the frontend and backend.
### 2- Creating a custom bridge network.
### 3- Running containers on different networks.
### 4- Verifying communication between containers.

---

## Step 1: Clone the Application

```bash
git clone https://github.com/Ibrahim-Adel15/Docker5.git
cd Docker5
```

---

## Step 2: Create Dockerfile for the Frontend

Create a file named `Dockerfile-f`.

```dockerfile
FROM python

WORKDIR /app

COPY frontend/requirements.txt .

RUN pip install  -r requirements.txt

COPY frontend/app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

Build the frontend image.

```bash
docker build -t app-front:v1 -f Dockerfile-f .
```

---

## Step 3: Create Dockerfile for the Backend

Create a file named `Dockerfile-b`.

```dockerfile
FROM python

WORKDIR /app

COPY backend/ .

RUN pip install flask

EXPOSE 5000

CMD ["python", "app.py"]
```

Build the backend image.

```bash
docker build -t app-back:v1 -f Dockerfile-b .
```

---

## Step 4: Create a Custom Docker Network

Create a bridge network named **ivolve-network** with subnet **192.168.10.0/24**.

```bash
docker network create \
--driver bridge \
--subnet 192.168.10.0/24 \
ivolve-network
```



---

## Step 5: Run the Backend Container

```bash
docker run -d \
--name backend \
--network ivolve-network \
app-back:v1
```

Verify its IP address.

```bash
docker inspect backend | grep -i IPAddress
```

output:

```text
"IPAddress": "192.168.10.2"
```

---

## Step 6: Run the First Frontend Container

```bash
docker run -d \
--name frontend1 \
--network ivolve-network \
-p 8080:5000 \
app-front:v1
```

Verify its IP address.

```bash
docker inspect frontend1 | grep -i IPAddress
```

output:

```text
"IPAddress": "192.168.10.3"
```

---

## Step 7: Run Another Frontend Container Using the Default Bridge Network

```bash
docker run -d \
--name frontend2 \
-p 8081:5000 \
app-front:v1
```

Verify its IP.

```bash
docker inspect frontend2 | grep -i IPAddress
```

Example output:

```text
"IPAddress": "172.17.0.2"
```

---

## Step 8: Verify Communication Between Containers

Verify **frontend1** can communicate with the backend.

```bash
curl localhost:8080
```

output:

```text
Frontend received: Hello from Backend!
```

Verify **frontend2** cannot communicate with the backend.

```bash
curl localhost:8081
```

output:

```text
Could not connect to backend.
```



---

## Step 9: Remove Containers

```bash
docker stop frontend1 frontend2 backend

docker rm frontend1 frontend2 backend
```

---

## Step 10: Delete the Custom Network

```bash
docker network rm ivolve-network
```

Verify it has been removed.

```bash
docker network ls
```

---