# Lab 03 - Run Java Spring Boot Application in a Docker Container

## Objective

Containerize a Java Spring Boot application using Docker by writing a Dockerfile, building the application inside the image using Maven, and running the generated JAR file.

---

## Repository

```sh
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
```



---

## Dockerfile

```dockerfile
FROM maven:3.9.9-eclipse-temurin-17

WORKDIR /app

COPY . .

RUN mvn package

EXPOSE 8080

CMD ["java", "-jar", "target/demo-0.0.1-SNAPSHOT.jar"]
```

---

## Build the Docker Image

```sh
docker build -t app1 .
```

---

## Verify the Image

```sh
docker images
```

**Expected Output**

```text
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
app1         latest    cacff8f4da0f   a few seconds    670MB
```

> **Note:** notice that the image size is 670MB copare it later with the next lab

---

## Run the Container

```sh
docker run -d --name container1 -p 8080:8080 app1
```



---

## Verify the Running Container

```sh
docker ps
```

**Expected Output**

```text
CONTAINER ID   IMAGE   COMMAND                  STATUS
3e8d5e7e6ab8   app1    "java -jar target/..."   Up
```

---

## Test the Application

Using curl:

```text
curl localhost:8080
```



**Expected Output**

```text
Hello from Dockerized Spring Boot!
```


---

## Stop the Container

```sh
docker stop container1
```

**Expected Output**

```text
container1
```

---

## Remove the Container

```sh
docker rm container1
```

**Expected Output**

```text
container1
```

---

## Verify Removal

```sh
docker ps -a
```

**Expected Output**

```text
CONTAINER ID   IMAGE   COMMAND   STATUS
```

The `container1` entry should no longer appear.

---
