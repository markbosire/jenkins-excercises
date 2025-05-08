# Concurrent Multibranch Pipeline with Lockable Resources

## Project repository

[https://github.com/markbosire/concurrent-multibranch.git](https://github.com/markbosire/concurrent-multibranch.git)

This project demonstrates how to implement concurrent builds in Jenkins while ensuring builds from different branches don't conflict, using:
- **Multibranch Pipelines** (automatic branch discovery)
- **Lockable Resources** (branch-specific resource locking)

## Problem Solved
**How can we enable concurrent builds but only if the builds are on different branches?**

This implementation allows:
- Simultaneous builds on different branches (dev, prod)
- Blocked concurrent builds on the same branch
- Resource safety through branch-specific locks

## Solution Architecture

### 1. Multibranch Pipeline
Automatically discovers and manages branches with individual Jenkinsfiles. Each branch gets its own build pipeline.

### 2. Lockable Resources
We use two dedicated lockable resources:
- `lock1` for dev branch
- `lock2` for prod branch

```groovy
// Jenkinsfile example (dev branch)
lock("lock1") {
    // Critical build steps here
}
```

## Implementation

### Prerequisites
1. Jenkins with:
   - Multibranch Pipeline plugin
   - Lockable Resources plugin
2. GitHub repository with:
   - `dev` and `prod` branches
   - Jenkinsfile in each branch

### Setup Steps

1. **Configure Lockable Resources**:
   - Go to *Manage Jenkins > System Configuration*
   - Add resources named `lock1` and `lock2`

2. **Create Multibranch Pipeline**:
   ```jenkins
   New Item > Multibranch Pipeline
   Branch Sources > Add GitHub repository
   Build Configuration > By Jenkinsfile
   ```

3. **Branch-specific Jenkinsfiles**:

`dev` branch Jenkinsfile:
```groovy
pipeline {
    agent any
    stages {
        stage("Build") {
            steps {
                lock("lock1") {
                    echo 'Dev build using lock1'
                    sh 'sleep 10' // Simulate work
                }
            }
        }
    }
}
```

`prod` branch Jenkinsfile:
```groovy
pipeline {
    agent any
    stages {
        stage("Build") {
            steps {
                lock("lock2") {
                    echo 'Prod build using lock2'
                    sh 'sleep 10' // Simulate work
                }
            }
        }
    }
}
```

## Behavior Examples

| Scenario | Result |
|----------|--------|
| Dev build starts | Acquires `lock1` |
| Another dev build starts | Waits for `lock1` |
| Prod build starts while dev is running | Acquires `lock2` (runs concurrently) |
| Two prod builds attempt to run | Second waits for `lock2` |

## Benefits

1. **Maximized Resource Utilization**:
   - Different branches can build simultaneously
   - No unnecessary queueing for unrelated work

2. **Branch Isolation**:
   - Dev and prod builds can't block each other
   - Critical branches get dedicated resources

3. **Safety Guarantees**:
   - Never more than one build per branch
   - Clear resource ownership during execution

## Troubleshooting

1. **Builds not running concurrently**:
   - Verify lock names match exactly
   - Check for typos in Jenkinsfiles
   - Confirm resources exist in Jenkins

2. **Locks not releasing**:
   - Check for build failures that might skip unlock
   - Manually release locks in Jenkins interface if needed

## Advanced Configuration

For more complex scenarios:
```groovy
lock(resource: 'lock1', inversePrecedence: true) {
    // Lets newer builds take precedence
}
```

Or use resource labels for dynamic allocation:
```groovy
lock(label: 'dynamic-locks', quantity: 1) {
    // Takes any available lock from pool
}
```
