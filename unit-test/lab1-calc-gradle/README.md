# Lab 01 - Building and Packaging Java Application with Gradle

## Objective

Learn how to build, test, package, and run a Java application using **Gradle**.

---

## required packages

- Java 17
- Gradle
- Git

---

## The main repo 

```bash
git clone https://github.com/Ibrahim-Adel15/calculator-gradle.git
```

## steps : 

- Installed Gradle
- Cloned the calculator project
- Executed unit tests
- Built the application
- Generated the JAR artifact
- Ran the application
- Verified that the application works successfully



# Run Unit Tests

```bash
gradle test
```
## you should see this :
```bash
BUILD SUCCESSFUL in 14s
3 actionable tasks: 3 executed
```

# Build the Application

```bash
gradle build
```
## if everythig is ok you should see this: 
```bash
BUILD SUCCESSFUL in 2s
7 actionable tasks: 4 executed, 3 up-to-date

```

## Generated Artifact

```text
build/libs/calculator.jar
```

# Run the Application

```bash
java -jar build/libs/calculator.jar
```

---
