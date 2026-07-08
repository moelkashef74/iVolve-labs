# Lab 01 - Building and Packaging Java Application with Gradle

## required packages

- Java 17
- Gradle
- Git

---
## The main repo 

```bash
git clone https://github.com/Ibrahim-Adel15/calculator-gradle.git
```


# Step 1 : Run Unit Tests

```bash
gradle test
```
### output :
```bash
BUILD SUCCESSFUL in 14s
3 actionable tasks: 3 executed
```

#  Step 2 : Build the Application

```bash
gradle build
```
### output: 
```bash
BUILD SUCCESSFUL in 2s
7 actionable tasks: 4 executed, 3 up-to-date

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

