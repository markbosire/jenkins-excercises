# Jenkins Email Notification Pipeline

This project demonstrates a practical Jenkins pipeline that uses the Email-Ext plugin to send comprehensive build notifications using only default Debian/Linux commands.

## Pipeline Features

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'echo "📦 Building application..."'
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    echo "🧪 Running tests..."
                    # Generate dummy test results
                    echo "SUCCESS: Login test passed
                    FAILURE: Payment test failed
                    SUCCESS: Profile test passed" > test-results.txt
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh 'echo "🚀 Deploying to staging..."'
            }
        }
    }

    post {
        always {
            script {
                // Capture system diagnostics
                def systemInfo = sh(script: 'uname -a && df -h && free -m', returnStdout: true).trim()
                def networkInfo = sh(script: 'ip a show', returnStdout: true).trim()
                
                // Analyze test results
                def testReport = sh(script: 'cat test-results.txt', returnStdout: true).trim()
                def successCount = sh(script: 'grep -c SUCCESS test-results.txt || true', returnStdout: true).trim()
                def failureCount = sh(script: 'grep -c FAILURE test-results.txt || true', returnStdout: true).trim()

                // Build email content
                def emailBody = """
                🛠️ Job: ${env.JOB_NAME}
                #️⃣ Build: ${env.BUILD_NUMBER}
                📌 Status: ${currentBuild.currentResult}
                
                💻 System Health:
                ${systemInfo}

                🌐 Network Status:
                ${networkInfo}

                ✅ Successful Tests: ${successCount}
                ❌ Failed Tests: ${failureCount}

                📄 Test Results:
                ${testReport}

                📋 Full Logs: ${env.BUILD_URL}console
                """

                // Send comprehensive status email
                emailext(
                    to: 'team@example.com',
                    subject: "🔔 ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: emailBody,
                    attachLog: true,
                    compressLog: true
                )
            }
        }
    }
}
```

## Use Cases for Email Notifications

1. **Build Status Alerts**
   - Immediate notification when builds fail
   - Success confirmations for critical deployments

2. **System Monitoring**
   - Disk space warnings before builds
   - Memory usage alerts during resource-intensive jobs

3. **Test Result Reporting**
   - Summary of test passes/failures
   - Quick visibility into regression issues

4. **Deployment Verification**
   - Confirmation of deployment completion
   - Early warning of deployment failures

5. **Team Coordination**
   - Notify multiple team members of build status
   - Provide direct links to logs for debugging

## Configuration Guide

### 1. Install Required Plugins
- Email Extension Plugin
- Pipeline plugin (if not already installed)

### 2. Configure Gmail SMTP Settings

1. Go to **Manage Jenkins** → **Configure System**
2. Scroll to **Extended E-mail Notification** section
3. Configure SMTP settings:

```
SMTP server: smtp.gmail.com
SMTP Port: 587
Use SSL: ☑ checked
Use TLS: ☑ checked
SMTP Username: your.email@gmail.com
SMTP Password: [your app password - see note below]
Charset: UTF-8
```

4. Set default recipient(s) in "Default Recipients" field
5. Click "Test configuration" to verify

**Important Note for Gmail:**
- You'll need to generate an "App Password" if using 2FA:
  1. Go to your Google Account → Security
  2. Under "Signing in to Google", select "App Passwords"
  3. Generate a password for "Mail" application
  4. Use this password in Jenkins configuration

- Alternatively, enable "Less secure app access" if not using 2FA

### 3. Pipeline Customization Options

1. **Recipients**: Change `team@example.com` to your actual email(s)
2. **Content**: Modify the email body template as needed
3. **Attachments**: Adjust `attachLog` and `compressLog` settings
4. **Diagnostics**: Add more system commands as required

## Best Practices

1. **Security**:
   - Use project-specific email distribution lists
   - Consider email content sensitivity (avoid secrets in logs)

2. **Performance**:
   - Limit attachment sizes for large log files
   - Be mindful of command output sizes

3. **Maintenance**:
   - Keep email templates clean and readable
   - Update recipients list as team changes

4. **Testing**:
   - Verify email delivery to all required recipients
   - Test both success and failure scenarios

This pipeline provides a solid foundation for build notifications that can be easily extended to meet specific project requirements while maintaining simplicity and using only standard Linux tools.
