# Jenkins Shared Library Project

## Overview
This repository contains a reusable Jenkins Shared Library designed to standardize CI/CD pipelines across projects. The library includes common build steps, logging utilities, and configuration templates to promote consistency and reduce boilerplate code in Jenkinsfiles.

## Why Shared Libraries Matter

### Key Benefits
1. **Code Reusability**  
   - Eliminate copy-paste of common pipeline logic
   - Maintain single sources of truth for build/deploy operations

2. **Standardization**  
   - Enforce organizational best practices
   - Ensure consistent tooling and versions across teams

3. **Maintainability**  
   - Update pipeline logic in one place
   - Simplify troubleshooting and auditing

4. **Security**  
   - Centralize credential handling
   - Enforce approved build methodologies

5. **Velocity**  
   - Reduce pipeline setup time from days to minutes
   - Enable self-service for development teams

## Library Structure
```
src/            # Groovy classes (business logic)
  com/devops/utils/Logger.groovy
vars/           # Pipeline steps
  buildApp.groovy
resources/      # Configuration templates
  configs/app-config.json
```

## Configuration in Jenkins

### 1. Global Configuration
1. Navigate to **Manage Jenkins → System Configuration → Global Pipeline Libraries**
2. Add new library with these settings:
   - **Name**: `my-shared-library` (must match @Library reference)
   - **Default Version**: `main` (or your preferred branch/tag)
   - **Retrieval Method**: Modern SCM
   - **Source Code Management**: Git
     - Project Repository: `https://github.com/markbosire/jenkins-shared-library-repo.git`
   - **Credentials**: Add if repository is private
3. Recommended options:
   - ☑ Allow default version to be overridden
   - ☑ Include @Library changes in job recent changes

### 2. Pipeline Configuration
In your Jenkinsfile:
```groovy
@Library('my-shared-library@main') _  // _ is required for imports

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Use library methods
                buildApp(appName: 'my-service', version: '1.0.0')
            }
        }
    }
}
```

### 3. Security Configuration (Optional)
For enterprise security:
1. **Script Approval**:
   - Approve methods under **Manage Jenkins → In-process Script Approval**
2. **Library Caching**:
   - Configure TTL under **Manage Jenkins → Configure System → Pipeline Libraries**
3. **Access Control**:
   - Set permissions via **Manage Jenkins → Security → Matrix Authorization**

## Testing Your Configuration
1. Create a test pipeline job
2. Paste the sample Jenkinsfile above
3. Run the pipeline - you should see:
   ```
   [INFO] [TIMESTAMP]: Building my-service with version 1.0.0
   ```

## Shared Library Repository
[https://github.com/markbosire/jenkins-shared-library-repo](https://github.com/markbosire/jenkins-shared-library-repo)

## Best Practices Implemented
✔ Pipeline-as-code standards  
✔ Environment-aware execution  
✔ Auditable logging  
✔ Safe parameter handling  
✔ Integration with common tools (Maven, Docker, K8s)  
✔ Version-controlled library updates  

