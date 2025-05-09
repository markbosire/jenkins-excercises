# Jenkins Script Console: Job Management Script

## Overview
This script allows administrators to stop all running builds for a specified Jenkins job using the Script Console. It's particularly useful for emergency situations or maintenance windows when you need to quickly halt executions of a problematic or resource-intensive job.

## Script Purpose
```groovy
def jobName = "remove"
def job = Jenkins.instance.getItemByFullName(jobName)

if (job) {
    job.builds.each { build ->
        if (build.isBuilding()) {
            println "Stopping build #${build.number}"
            build.doStop()
        }
    }
} else {
    println "Job ${jobName} not found"
}
```

## Key Use Cases for Script Console
1. **Emergency Job Termination** - Immediately stop all running instances of a job
2. **Bulk Operations** - Perform actions across multiple jobs/builds without UI clicks
3. **Troubleshooting** - Inspect job states or configurations programmatically
4. **Automation** - Script complex maintenance tasks that aren't available in the UI
5. **System Recovery** - Fix issues when the regular UI might be unresponsive

## How to Use
1. Access Jenkins Script Console (`Manage Jenkins` > `Script Console`)
2. Modify the `jobName` variable to target your specific job
3. Paste and execute the script
4. Review output for confirmation of stopped builds

## Important Notes
- Requires ADMIN permissions
- Changes are immediate and irreversible
- Always test scripts in a non-production environment first
- Consider backing up your Jenkins instance before running system scripts

## Safety Recommendations
- Double-check the job name before execution
- Limit script console access to trusted administrators
- Log all script console activities for audit purposes
