# Jenkins Pipeline for Stage-Specific Log Archiving

This Jenkins pipeline defines two stages (`Build` and `Test`) where logs are captured and stored in files, even when using ephemeral agents. If any stage fails, the log for that stage will be archived for future reference.

## Pipeline Overview

- **Agent**: The pipeline uses the `agent any` directive, which allows it to run on any available agent.
- **Stages**:
  - **Build**: This stage executes the build process and stores any error logs into a file (`build_${BUILD_NUMBER}.error.log`). In case of a failure, the error log is archived.
  - **Test**: This stage runs the tests and captures any error logs into a file (`test_${BUILD_NUMBER}.error.log`). Similarly, if it fails, the error log is archived.
- **Post Actions**:
  - For both stages, if the stage fails, the associated log file is archived for later retrieval.
  - After all stages have completed, the workspace is cleaned up to ensure no leftover files.

## How It Works

1. **Log File Creation**: In each stage, a shell script captures standard error (`stderr`) using `bash` and redirects it into a log file. This is achieved by using `tee` to write to both the terminal and the log file at the same time.

2. **Error Handling**: If a stage fails (e.g., during the `Build` or `Test` steps), the log file is automatically archived using `archiveArtifacts`. This ensures that even when an ephemeral agent is used, the log files are preserved and accessible for debugging.

3. **Workspace Cleanup**: The `cleanWs()` command in the `post` block ensures that any temporary files created during the pipeline execution are removed after all stages are finished, regardless of whether the pipeline succeeds or fails.

## Example Pipeline

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    // Multiline shell script with bash and redirection for stderr
                    sh '''
                        bash -c "
                        exec 2> >(tee build_${BUILD_NUMBER}.error.log >&2)
                        echo 'Building...'
                        "
                    '''
                }
            }
            post {
                failure {
                    // Archive the log if the stage fails
                    archiveArtifacts artifacts: "build_${BUILD_NUMBER}.error.log", allowEmptyArchive: true
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Multiline shell script with bash and redirection for stderr
                    sh '''
                        bash -c "
                        exec 2> >(tee test_${BUILD_NUMBER}.error.log >&2)
                        echo 'Testing...'
                        python test.py
                        "
                    '''
                }
            }
            post {
                failure {
                    // Archive the log if the stage fails
                    archiveArtifacts artifacts: "test_${BUILD_NUMBER}.error.log", allowEmptyArchive: true
                }
            }
        }
    }

    post {
        always {
            // Clean up the workspace after all stages, regardless of the outcome
            cleanWs()
        }
    }
}
```

## Key Features

- **Ephemeral Agent Compatibility**: Logs are captured and stored locally within the pipeline, ensuring that even if the agent is ephemeral (temporary), the logs are still available for later inspection.
- **Error Log Archiving**: Each stage's error logs are archived if the stage fails, ensuring you have access to critical diagnostic information for troubleshooting.
- **Automatic Cleanup**: The pipeline ensures that no unnecessary files are left in the workspace after the pipeline runs, keeping the environment clean and organized.

## Requirements

- Jenkins must be set up to allow the use of `archiveArtifacts` and handle `sh` steps.
- Ensure that the `BUILD_NUMBER` variable is set correctly for log file naming.
```

This `README.md` file provides a detailed explanation of how the pipeline handles log files, archives them on failure, and cleans up after completion. It includes the pipeline code and descriptions of key features to ensure clarity.
