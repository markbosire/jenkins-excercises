# Connecting Jenkins to the Self-Hosted Git Server

This guide shows how to configure Jenkins to connect to the self-hosted Git server set up [in this tutorial](./Gitserversetup.md).

## Requirements
- Jenkins installed
- SSH access between Jenkins and the Git server
- Jenkins user has SSH permissions

## 1. Set Up SSH Authentication

First, on your Jenkins server, switch to the Jenkins user:

```bash
sudo su - jenkins
```

Generate an SSH key pair if it doesn't exist:

```bash
ssh-keygen -t rsa -b 4096 -C "jenkins@yourdomain.com"
```

Press Enter to accept the defaults (no passphrase needed).

Next, copy the public key to the Git server (inside the Vagrant box):

```bash
ssh-copy-id git@<vagrant-ip-address>
```

If ssh-copy-id is not installed, you can manually copy it:

```bash
cat ~/.ssh/id_rsa.pub
```

Then SSH into the Git server, and append it to /home/git/.ssh/authorized_keys.

Example manually:

```bash
ssh vagrant@<vagrant-ip-address>
sudo su - git
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
# Paste the public key here
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

## 2. Configure Jenkins Git Connection Using Known Hosts Method

In Jenkins:

1. Go to Manage Jenkins > Security > Git Host Key Verification Configuration
2. Select "Known hosts file" option
3. Jenkins will use the standard SSH known_hosts files to verify Git server identities

## 3. Create a Jenkins Job

When setting up a Jenkins pipeline or freestyle job:

1. In Source Code Management, choose Git
2. For Repository URL, you can simply use:
   ```
   git@<vagrant-ip-address>:/home/git/repos/myrepo.git
   ```
3. No additional credentials configuration is needed since we're using the known hosts verification method

This ensures Jenkins can authenticate over SSH and clone your self-hosted Git repository securely.
