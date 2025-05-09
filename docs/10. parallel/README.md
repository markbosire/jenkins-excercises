# Jenkins Parallel Execution Pipeline

This pipeline demonstrates parallel stage execution in Jenkins, showcasing how to:
1. Perform initial setup
2. Run multiple tasks concurrently
3. Merge and report results

## Pipeline Stages

### 1. Parallel Processing Stage (Key Feature)
Three independent stages run simultaneously:

#### a) Analyze Files
- Performs text processing operations:
  - Line count of data file
  - Extracts numbers containing "5"
  - Shows frequency of number prefixes

#### b) System Checks
- Gathers system information:
  - Hostname
  - CPU core count
  - Memory usage
  - Disk space

#### c) Data Processing
- Processes data without `bc` command:
  - Counts total lines
  - Identifies unique numbers
  - Calculates sum using shell arithmetic

### 2. Merge Results Stage
- Combines outputs from parallel stages
- Creates comprehensive report with:
  - Timestamp
  - CPU information
  - File statistics
  - Parallel stage outputs
- Archives `report.txt`

## Parallel Execution Benefits

1. **Faster Builds**: Independent tasks run concurrently
2. **Resource Efficiency**: Better utilization of agent resources
3. **Isolated Failures**: One failing stage doesn't block others
4. **Clear Visualization**: Jenkins UI shows parallel branches clearly

## Use Cases for Parallel Stages

- Running tests against different environments
- Performing simultaneous build/deploy tasks
- Gathering system metrics while processing data
- Executing independent quality checks
- Building multiple components of a system

## Cleanup
The workspace is automatically cleaned after pipeline completion (in `post.always`).

## How to Run
1. Create a new Pipeline job in Jenkins
2. Paste this script in the Pipeline script section
   ```
   pipeline {
    agent any
    
    stages {
        stage('Setup') {
            steps {
                sh '''
                    mkdir -p artifacts
                    echo "Linux" > artifacts/os.txt
                    seq 1 100 > artifacts/data.txt
                    md5sum artifacts/* > artifacts/checksums.md5 2>/dev/null || true
                '''
            }
        }

        stage('Parallel Processing') {
            parallel {
                stage('Analyze Files') {
                    steps {
                        sh '''
                            echo "=== FILE ANALYSIS ==="
                            wc -l artifacts/data.txt
                            grep "5" artifacts/data.txt | head -5
                            cut -c1-3 artifacts/data.txt | sort | uniq -c
                        '''
                    }
                }

                stage('System Checks') {
                    steps {
                        sh '''
                            echo "=== SYSTEM CHECK ==="
                            echo "Host: $(hostname)"
                            echo "Cores: $(nproc)"
                            echo "Memory: $(free -h | awk '/Mem:/{print $2}' || echo "N/A")"
                            df -h | grep -v tmpfs || echo "Disk info unavailable"
                        '''
                    }
                }

                stage('Data Processing') {
                    steps {
                        sh '''
                            echo "=== DATA PROCESSING (NO BC) ==="
                            echo "Total lines: $(wc -l < artifacts/data.txt)"
                            echo "Unique numbers: $(sort -u artifacts/data.txt | wc -l)"
                            
                            # Alternative to 'bc' for summing numbers
                            sum=0
                            while read num; do
                                sum=$((sum + num))
                            done < artifacts/data.txt
                            echo "Sum of numbers: $sum" > artifacts/sum.txt
                            cat artifacts/sum.txt
                        '''
                    }
                }
            }
        }

        stage('Merge Results') {
            steps {
                sh '''
                    echo "=== FINAL REPORT ===" > report.txt
                    date >> report.txt
                    echo "\nCPU Info:" >> report.txt
                    grep -m1 "model name" /proc/cpuinfo >> report.txt 2>/dev/null || echo "CPU info unavailable" >> report.txt
                    echo "\nFile Stats:" >> report.txt
                    wc artifacts/* >> report.txt 2>/dev/null || true
                    
                    echo "\n--- Parallel Stage Outputs ---" >> report.txt
                    echo "System Cores: $(nproc)" >> report.txt
                    echo "Data Sum: $(cat artifacts/sum.txt 2>/dev/null)" >> report.txt
                    
                    cat report.txt
                '''
                archiveArtifacts artifacts: 'report.txt'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
   ```
4. Run the job and observe parallel stage execution
