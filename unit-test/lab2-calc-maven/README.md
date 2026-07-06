# Lab 02 - Building and Packaging Java Application with Maven

## Objective

Learn how to build, test, package, and run a Java application using **Maven**.

---

## required packages

- Java 17 or the latest version
- Maven
- Git

---

## The main repo 

```bash
https://github.com/Ibrahim-Adel15/calculator-maven.git
```

## steps : 

- Installed Maven
- Cloned the calculator project
- Executed unit tests
- Built the application
- Generated the JAR artifact
- Ran the application
- Verified that the application works successfully



# Run Unit Tests

```bash
mvn test
```
This will:
Compile the source code.
Compile the test code.
Run all tests under:
```bash
src/test/java
```
## you should see this :
```bash
[INFO]
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.calculator.CalculatorTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.120 s -- in com.example.calculator.CalculatorTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  36.300 s
[INFO] Finished at: 2026-07-06T19:34:49+03:00
[INFO] ------------------------------------------------------------------------
```

# Build the Application

```bash
mvn package
```
This command will:

Compile the project.
Run unit tests.
Package the application into a JAR

The generated artifact will be located in:
```bash
target/
```
## if everythig is ok you should see this: 
```bash
[INFO] Building jar: /home/mo/iVolve-labs/unit-test/lab2-calc-maven/target/calculator.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  13.150 s
[INFO] Finished at: 2026-07-06T19:38:50+03:00

```

## Generated Artifact

```text
target/calculator.jar
```

# Run the Application

```bash
java -jar build/libs/calculator.jar
```

---
