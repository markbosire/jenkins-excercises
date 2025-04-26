# Jenkins Rsync Pipeline Documentation

## Overview
This Jenkins pipeline automates the process of synchronizing multiple Git branches to corresponding directories on a server using rsync. The pipeline checks out a repository, installs rsync if needed, and then syncs each specified branch to its own directory.

## Pipeline Configuration

### Environment Variables
- `GIT_REPO`: GitHub repository URL (`https://github.com/markbosire/jenkins-rsync.git`)
- `BASE_DIR`: Base directory for synced content (`/home/vagrant/rscync`)
- `GIT_CREDENTIALS`: Jenkins credentials ID for Git access (`gittoken`)
- `DEFAULT_BRANCH`: Default branch to check out first (`dev`)

### Stages

1. **Checkout Repository**
   - Checks out the specified Git repository using the provided credentials
   - Uses the default branch specified in `DEFAULT_BRANCH`

2. **Check Jenkins User**
   - Outputs the current user running the Jenkins job (for debugging)

3. **Install rsync**
   - Installs rsync using `apt` (requires sudo privileges)

4. **Sync Branches**
   - Processes three branches (`prod`, `dev`, `test`)
   - For each branch:
     - Checks out the branch
     - Creates the target directory if it doesn't exist
     - Uses rsync to sync all files (except `.git/`) to the corresponding directory under `BASE_DIR`
     - Uses `--delete` flag to remove files in destination not present in source
     - Uses `-avz` flags for archive mode, verbose output, and compression

### Post-Actions
- Always cleans up the workspace by deleting the working directory

### Repo link
[repo](https://github.com/markbosire/jenkins-rsync)
