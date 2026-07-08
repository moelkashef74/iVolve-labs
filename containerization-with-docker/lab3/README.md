# Lab 03 - Run Java Spring Boot Application in a Docker Container


## Step 1: Clone the Application Source Code

```sh
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
cd Docker-1
```

---

## Step 2: Create the Dockerfile


```dockerfile
FROM maven:3.9.9-eclipse-temurin-17

WORKDIR /app

COPY . .

RUN mvn package

EXPOSE 8080

CMD ["java", "-jar", "target/demo-0.0.1-SNAPSHOT.jar"]
```


---

## Step 3: Build the Docker Image

```sh
docker build -t app1 .
```

---

## Step 4: Verify the Image

```sh
docker images
```

output:

```text
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
app1         latest    cacff8f4da0f   a few seconds    670MB
```

> **Note:** Record the image size (**670 MB**) and compare it with the optimized image created in the next lab.

---

## Step 5: Run a Container from the Image

```sh
docker run -d --name container1 -p 8080:8080 app1
```

---

## Step 6: Verify the Running Container

```sh
docker ps
```

Expected output:

```text
CONTAINER ID   IMAGE   COMMAND                  STATUS
3e8d5e7e6ab8   app1    "java -jar target/..."   Up
```

---

## Step 7: Test the Application

Using curl:

```sh
curl localhost:8080
```

Expected output:

```text
Hello from Dockerized Spring Boot!
```

---

## Step 8: Stop the Container

```sh
docker stop container1
```

Expected output:

```text
container1
```

---

## Step 9: Remove the Container

```sh
docker rm container1
```

Expected output:

```text
container1
```

---

## Step 10: Verify the Container Has Been Removed

```sh
docker ps -a
```

Expected output:

```text
CONTAINER ID   IMAGE   COMMAND   STATUS
```

The `container1` entry should no longer appear.
