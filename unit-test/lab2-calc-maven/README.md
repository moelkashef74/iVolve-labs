# Lab 02 - Building and Packaging Java Application with Maven

## required packages

- Java 17
- Maven 
- Git
---
## The main repo 

```bash
git clone https://github.com/Ibrahim-Adel15/calculator-gradle.git
```

# Step 1 : Run Unit Tests

```bash
mvn test
```
### output :
```bash
[INFO] Running com.example.calculator.CalculatorTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.076 s -- in com.example.calculator.CalculatorTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.417 s
[INFO] Finished at: 2026-07-08T13:50:53+03:00

```

#  Step 2 : Build the Application

```bash
mvn package
```
this step will :-
Compile the project. ,
Run unit tests. ,
Package the application into a JAR

### output: 
```bash
 T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.calculator.CalculatorTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.075 s -- in com.example.calculator.CalculatorTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO]
[INFO] --- jar:3.4.1:jar (default-jar) @ calculator ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.057 s
[INFO] Finished at: 2026-07-08T13:51:45+03:00


```

##  **note**:- Step 2 will generate Artifact

```text
build/libs/calculator.jar
```

# Step 3 : Run the Application

```bash
java -jar build/libs/calculator.jar
```
### output: 
```bash
Enter first number: 16
Enter second number: 8
a = 16.0, b = 8.0
Sum: 24.0
Subtract: 8.0
Multiply: 128.0
Divide: 2.0

```

---

