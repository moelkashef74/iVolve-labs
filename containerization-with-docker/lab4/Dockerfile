FROM amazoncorretto:17-alpine
WORKDIR /app
COPY pom.xml .
COPY target/demo-0.0.1-SNAPSHOT.jar .
CMD ["java", "-jar", "demo-0.0.1-SNAPSHOT.jar"]
EXPOSE 8080
