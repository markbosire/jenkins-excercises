# Self-Hosted Git Server Setup (Vagrant)

This guide will show you how to set up a basic self-hosted Git server inside a Vagrant box and create a simple Git repository.

## Requirements
- Vagrant
- VirtualBox
- Git installed on your machine

## 1. Create and Start the Vagrant Box

First, create a `Vagrantfile` with the following contents:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/debian11"
  config.vm.box_version = "4.3.12"
  config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
  end
  
  # Assign a public IP to simulate a cloud instance i manually added afree ip you can check which free ips are in the network u used an your network adapters name

  config.vm.network "public_network", ip: "192.168.100.204", bridge: "Intel(R) Dual Band Wireless-AC 8265"
  
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y git
    useradd -m git
    mkdir -p /home/git/repos
    chown git:git /home/git/repos
  SHELL
end
```

Then, run:

```bash
vagrant up
vagrant ssh
```

## 2. Set Up the Git Server

SSH into the machine if you haven't:

```bash
vagrant ssh
```

Switch to the git user:

```bash
sudo su - git
```

Create a simple bare repository:

```bash
cd ~/repos
git init --bare myrepo.git
```

This repository (myrepo.git) will act as your remote repository.

## 3. Access the Git Server

To clone the repository from your host machine (outside of Vagrant), you'll need the Vagrant box's IP address. You can find it by running:

```bash
vagrant ssh-config
```

Then you can clone using:

```bash
git clone ssh://git@<vagrant-ip-address>/home/git/repos/myrepo.git
```
