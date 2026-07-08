# Lab 04 - Run Java Spring Boot Application in a Docker Container

## Objective

Containerize a Java Spring Boot application using Docker by writing a Dockerfile, running the generated JAR file that built locally

---

## Repository

```sh
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
```



---

## Dockerfile

```dockerfile
FROM amazoncorretto:17-alpine

WORKDIR /app

COPY pom.xml .

COPY target/demo-0.0.1-SNAPSHOT.jar .

CMD ["java", "-jar", "demo-0.0.1-SNAPSHOT.jar"]

EXPOSE 8080

```

---

## Build the Docker Image

```sh
docker build -t java-app:v2 .
```

---

## Verify the Image

```sh
docker images
```

**Expected Output**

```text
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
java-app:v2                     c3093f76b102        490MB          170MB 
```

> **Note:** notice here the size of the image is reduced to 49MB remember when we built the app using dockerfile the size was 670MB

---

## Run the Container

```sh
docker run -d --name container2 -p 8080:8080 java-app:v2
```



---

## Verify the Running Container

```sh
docker ps
```

**Expected Output**

```text
CONTAINER ID   IMAGE   COMMAND                  STATUS
3e8d5e7e6ab8   java-app:v2    "java -jar target/..."   Up
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
docker rm container2
```

**Expected Output**

```text
container2
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

The `container2` entry should no longer appear.

---
