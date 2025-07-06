# Maven Quick Reference Guide

## What is Maven?
Maven is a build automation and project management tool for Java projects. It uses XML configuration files (pom.xml) to manage dependencies, build processes, and project structure.

## Project Structure
```
my-project/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── target/
```

## Core Concepts

### POM (Project Object Model)
The `pom.xml` file contains project configuration, dependencies, and build settings.

### Coordinates
Every Maven project is identified by:
- **groupId**: Organization identifier (e.g., `com.example`)
- **artifactId**: Project name (e.g., `my-app`)
- **version**: Project version (e.g., `1.0.0`)

### Dependencies
External libraries your project needs to function.

### Repositories
Remote locations where Maven downloads dependencies (Maven Central, custom repos).

## Basic POM Structure
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

## Essential Commands

### Project Creation
```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

### Build Lifecycle
```bash
mvn clean          # Clean target directory
mvn compile        # Compile source code
mvn test           # Run tests
mvn package        # Create JAR/WAR
mvn install        # Install to local repository
mvn deploy         # Deploy to remote repository
```

### Common Commands
```bash
mvn clean compile                    # Clean and compile
mvn clean test                       # Clean and test
mvn clean package                    # Clean and package
mvn clean install                    # Clean, compile, test, package, install
mvn dependency:tree                  # Show dependency tree
mvn dependency:resolve               # Download dependencies
mvn help:effective-pom               # Show effective POM
mvn versions:display-dependency-updates  # Check for dependency updates
```

## Dependency Scopes
- **compile**: Default scope, available in all classpaths
- **test**: Only available during testing
- **runtime**: Not needed for compilation, but required for execution
- **provided**: Provided by runtime environment (e.g., servlet-api)
- **system**: Similar to provided, but JAR is explicitly provided
- **import**: Used with dependency management in POMs

## Dependency Management
```xml
<dependencyManagement>
    <dependencies
