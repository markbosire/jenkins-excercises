# HelloKaniko Project

## Overview

HelloKaniko is a demonstration project that showcases how to build and deploy containerized Java applications using Jenkins pipelines with Kaniko on ephemeral Kubernetes agents.

## Project Repository

[https://github.com/markbosire/hello-kaniko](https://github.com/markbosire/hello-kaniko)

## Key Components

- **Java Application**: A simple Java application that demonstrates containerized deployment
- **Jenkins Pipeline**: CI/CD pipeline configured using a Jenkinsfile
- **Kaniko**: Tool for building container images within Kubernetes without requiring Docker daemon
- **Kubernetes**: Orchestration platform for running ephemeral Jenkins agents

## Why Ephemeral Kubernetes Agents?

### Advantages

1. **Scalability**: Kubernetes can automatically scale agents based on workload demand
2. **Isolation**: Each build job runs in its own isolated container
3. **Efficiency**: Resources are only consumed during active builds
4. **Consistency**: Every build starts with a clean environment
5. **Resource Optimization**: No need to maintain idle agents
6. **Flexibility**: Easily use different tools and environments for different builds


## Benefits of Using Kaniko

1. **No Docker Daemon**: Builds containers without requiring privileged access
2. **Security**: Runs without root privileges
3. **Kubernetes Native**: Designed to work within Kubernetes environments
4. **Cache Support**: Supports layer caching for faster builds
5. **Registry Integration**: Directly pushes to container registries

