# JUnit & JaCoCo Integration in Jenkins

This project demonstrates how to integrate JUnit testing and JaCoCo code coverage in a Jenkins CI/CD pipeline. The implementation includes parameterized tests and comprehensive coverage reporting with quality gates.

## Overview

The integration consists of:

1. **JUnit 5 tests** with parameterized testing capabilities
2. **JaCoCo** for code coverage analysis
3. **Jenkins pipeline** that builds, tests, and reports on both test results and coverage metrics

## Project Structure

```
project/
├── src/
│   ├── main/java/
│   │   └── com/example/
│   │       ├── Calculator.java
│   │       └── StringUtils.java
│   └── test/java/
│       └── com/example/
│           ├── CalculatorTest.java
│           └── StringUtilsTest.java
├── pom.xml
└── Jenkinsfile
```

## Prerequisites

- Jenkins server with the following plugins installed:
  - JUnit Plugin
  - JaCoCo Plugin
  - Coverage Plugin
  - Git Forensics Plugin (for modified lines coverage)
- Maven installed and configured in Jenkins

## Maven Dependencies

Your project needs the following dependencies in the `pom.xml` file:

```xml
<!-- JUnit Jupiter API -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>

<!-- JUnit Jupiter Engine -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>

<!-- JUnit Jupiter Params for parameterized tests -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.9.3</version>
    <scope>test</scope>
</dependency>

<!-- JaCoCo Plugin -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Jenkins Pipeline Explained

The Jenkins pipeline defines the CI/CD process with four main components:

### 1. Dependency Management

The pipeline ensures the JUnit Params dependency is available for parameterized tests:

```groovy
stage('Update Dependencies') {
    steps {
        // Script to check for and add junit-jupiter-params if missing
    }
}
```

### 2. Build Phase

```groovy
stage('Build') {
    steps {
        sh 'mvn clean compile'
    }
}
```

This stage compiles the code and installs dependencies without running tests.

### 3. Test Phase

```groovy
stage('Test') {
    steps {
        sh 'mvn test'
    }
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
        }
    }
}
```

The test phase:
- Runs all JUnit tests with the Maven Surefire plugin
- JaCoCo automatically collects coverage data during this phase
- The JUnit test results are saved for later reporting

### 4. Coverage Reporting

The pipeline uses the modern Coverage plugin (replacing the older JaCoCo plugin) for advanced coverage reporting:

```groovy
post {
    always {
        recordCoverage(
            tools: [
                [parser: 'JACOCO']
            ],
            id: 'jacoco-coverage',
            name: 'JaCoCo Coverage Report',
            sourceCodeRetention: 'EVERY_BUILD',
            qualityGates: [
                [threshold: 80.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
                [threshold: 70.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true],
                [threshold: 90.0, metric: 'LINE', baseline: 'MODIFIED_LINES', unstable: true]
            ],
            skipSymbolicLinks: true,
            sourceDirectories: [
                [path: 'src/main/java']
            ]
        )
        
        recordCoverage(
            tools: [
                [parser: 'JUNIT']
            ],
            id: 'junit-tests',
            name: 'JUnit Test Results'
        )
    }
}
```

## In-Depth Coverage Configuration Explanation

### JaCoCo Coverage Report

The configuration uses the modern `recordCoverage` step with the following details:

1. **Parser Configuration**: 
   ```groovy
   tools: [[parser: 'JACOCO']]
   ```
   This specifies JaCoCo as the coverage tool parser.

2. **Report Identification**:
   ```groovy
   id: 'jacoco-coverage',
   name: 'JaCoCo Coverage Report'
   ```
   Sets unique ID and display name for the coverage report.

3. **Source Code Retention**:
   ```groovy
   sourceCodeRetention: 'EVERY_BUILD'
   ```
   Preserves source code for each build to enable historical coverage analysis.

4. **Quality Gates**:
   ```groovy
   qualityGates: [
       [threshold: 80.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
       [threshold: 70.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true],
       [threshold: 90.0, metric: 'LINE', baseline: 'MODIFIED_LINES', unstable: true]
   ]
   ```
   These are the criteria that determine pipeline health:
   - 80% minimum line coverage across the project
   - 70% minimum branch coverage across the project
   - 90% minimum line coverage on modified lines (requires Git Forensics)
   - `unstable: true` means the build is marked unstable rather than failed when thresholds aren't met

5. **Source Directory Specification**:
   ```groovy
   sourceDirectories: [[path: 'src/main/java']]
   ```
   Explicitly defines which directories to analyze for coverage.

### JUnit Test Results

A separate `recordCoverage` step captures test results:
```groovy
recordCoverage(
    tools: [[parser: 'JUNIT']],
    id: 'junit-tests',
    name: 'JUnit Test Results'
)
```

This provides detailed test execution metrics including:
- Test counts (passed, failed, skipped)
- Test duration statistics
- Failure details and stack traces

## Interpreting Results

In the Jenkins UI, you'll see:

1. **Test Results Trend** - Shows test pass/fail rates over time
2. **Coverage Metrics** - Line, branch, and method coverage percentages
3. **Coverage Trends** - How coverage changes between builds
4. **Quality Gate Status** - Whether coverage meets the defined thresholds

## Troubleshooting

### Common Issues

1. **Missing Dependencies**: Ensure junit-jupiter-params is in your pom.xml
2. **JaCoCo Configuration**: Verify jacoco-maven-plugin is properly configured
3. **Source Paths**: Make sure sourceDirectories points to the correct location
