# Lab 05 - Build a Java Spring Boot Application Using a Multi-Stage Docker Build


## Step 1: Clone the Application Source Code

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
```

---

## Step 2: Create the Dockerfile

```dockerfile
# ---------- Builder Stage ----------
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

COPY . .

RUN mvn package

# ---------- Runtime Stage ----------
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## Step 3: Build the Docker Image

```bash
docker build -t java-app:v1 .
```

Expected output:

```text
[+] Building ...
...
Successfully built b3jht5fmjhgg
Successfully tagged java-app:v1
```

---

## Step 4: Verify the Image

```bash
docker images
```

Expected output:

```text
REPOSITORY   TAG    IMAGE ID       CREATED          SIZE
java-app     v1     b3jht5fmjhgg     A few seconds    ~250MB
```


## Step 5: Run a Container from the Image

```bash
docker run -d --name java-container -p 8080:8080 java-app:v1
```

---

## Step 6: Verify the Running Container

```bash
docker ps
```

Expected output:

```text
CONTAINER ID   IMAGE         COMMAND                  STATUS         PORTS
<CONTAINER_ID> java-app:v1   "java -jar app.jar"      Up             0.0.0.0:8080->8080/tcp
```

---

## Step 7: Test the Application

Using curl:

```bash
curl localhost:8080
```

Expected output:

```text
Hello from Dockerized Spring Boot!
```

---

## Step 8: Stop the Container

```bash
docker stop java-container
```
---

## Step 9: Remove the Container

```bash
docker rm java-container
```

