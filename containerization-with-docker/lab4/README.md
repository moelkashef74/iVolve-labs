# Lab 04 - Run Java Spring Boot Application in a Lightweight Docker Container

## Step 1: Clone the Application Source Code

```sh
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
cd Docker-1
```

---

## Step 2: Build the Application Locally

Build the Spring Boot application using Maven to generate the executable JAR file.

```sh
mvn package
```

Expected output:

```text
BUILD SUCCESS
```

The generated artifact will be located at:

```text
target/demo-0.0.1-SNAPSHOT.jar
```

---

## Step 3: Create the Dockerfile

```dockerfile
FROM amazoncorretto:17-alpine

WORKDIR /app

COPY pom.xml .

COPY target/demo-0.0.1-SNAPSHOT.jar .

EXPOSE 8080

CMD ["java", "-jar", "demo-0.0.1-SNAPSHOT.jar"]
```



## Step 4: Build the Docker Image

```sh
docker build -t java-app:v2 .
```

---

## Step 5: Verify the Image

```sh
docker images
```

Expected output:

```text
REPOSITORY   TAG    IMAGE ID       CREATED          SIZE
java-app     v2     c3093f76b102   A few seconds    170MB
```

> **Note:**  
> In **Lab 03**, the Docker image size was approximately **670 MB** because the image contained Maven, the source code, and all build dependencies.
>
> In this lab, the image size is reduced to approximately **170 MB** because it only contains the compiled JAR file and a lightweight Java runtime. This approach produces a much smaller production-ready image.

---

## Step 6: Run a Container from the Image

```sh
docker run -d --name container2 -p 8080:8080 java-app:v2
```


---

## Step 7: Verify the Running Container

```sh
docker ps
```

Expected output:

```text
CONTAINER ID   IMAGE         COMMAND                            STATUS
3e8d5e7e6ab8   java-app:v2   "java -jar demo-0.0.1-SNAPSHOT…"   Up
```

---

## Step 8: Test the Application

Using curl:

```sh
curl localhost:8080
```

Expected output:

```text
Hello from Dockerized Spring Boot!
```

---

## Step 9: Stop the Container

```sh
docker stop container2
```


## Step 10: Remove the Container

```sh
docker rm container2
```


---

## Step 11: Verify the Container Has Been Removed

```sh
docker ps -a
```

Expected output:

```text
CONTAINER ID   IMAGE   COMMAND   STATUS
```

The `container2` entry should no longer appear.
