# Docker-projects
# README: Docker Project - Setting up SSH Connection between Two Ubuntu Containers

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Project Overview](#project-overview)
4. [Steps to Set Up the Project](#steps-to-set-up-the-project)
   - [Step 1: Install Docker](#step-1-install-docker)
   - [Step 2: Create Docker Network](#step-2-create-docker-network)
   - [Step 3: Create Dockerfile for SSH Setup](#step-3-create-dockerfile-for-ssh-setup)
   - [Step 4: Build Docker Image](#step-4-build-docker-image)
   - [Step 5: Run Two Containers](#step-5-run-two-containers)
   - [Step 6: Configure SSH in Containers](#step-6-configure-ssh-in-containers)
   - [Step 7: SSH from One Container to Another](#step-7-ssh-from-one-container-to-another)
5. [Verification](#verification)
6. [Cleaning Up](#cleaning-up)
7. [Conclusion](#conclusion)

## Introduction

This project demonstrates how to establish an SSH connection between two Ubuntu containers using Docker. It is a practical exercise to learn how to manage Docker containers, set up SSH, and enable secure communication between isolated environments.

By the end of this project, you will have two Ubuntu containers running with SSH services enabled, allowing you to connect from one container to the other via SSH.

## Prerequisites

Before proceeding, ensure you have the following installed on your system:
- Docker (version 19.03 or later)
- Basic understanding of Docker commands
- SSH client (should be installed by default on most Unix-like systems)

For installation instructions, refer to the [official Docker documentation](https://docs.docker.com/get-docker/).

## Project Overview

In this project, we will:
1. Create two Ubuntu containers.
2. Install and configure SSH on both containers.
3. Connect from one container to the other using SSH.

The communication between containers will be done through a custom Docker network to facilitate easier connection using container names.

## Steps to Set Up the Project

### Step 1: Install Docker

If you haven’t installed Docker yet, use the following commands to install it on your Linux machine (for Ubuntu):

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce
```

Verify the installation:

```bash
docker --version
```

### Step 2: Create Docker Network

To allow the containers to communicate via SSH, create a custom Docker network:

```bash
docker network create ssh-network
```

### Step 3: Create Dockerfile for SSH Setup

We need a Dockerfile to define the base image, install SSH server, and configure it for each container.

Create a `Dockerfile` with the following content:

```Dockerfile
# Use the official Ubuntu base image
FROM ubuntu:20.04

# Install updates, SSH server, and other necessary packages
RUN apt-get update && apt-get install -y openssh-server sudo

# Create a directory for the SSH daemon
RUN mkdir /var/run/sshd

# Create a new user and set password for SSH access
RUN useradd -rm -d /home/ubuntuuser -s /bin/bash -g root -G sudo -u 1000 ubuntuuser && \
    echo 'ubuntuuser:password' | chpasswd

# Allow password authentication
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Expose port 22 for SSH
EXPOSE 22

# Start SSH service
CMD ["/usr/sbin/sshd", "-D"]
```

This Dockerfile does the following:
- Uses the official Ubuntu 20.04 image.
- Installs SSH server (`openssh-server`).
- Creates a user (`ubuntuuser`) with a password for SSH access.
- Configures the SSH daemon for password authentication and root login.
- Exposes port 22 for SSH access.

### Step 4: Build Docker Image

Now, build the Docker image using the Dockerfile:

```bash
docker build -t ubuntu-ssh .
```

### Step 5: Run Two Containers

Run two containers using the image created and connect them to the custom Docker network (`ssh-network`):

```bash
docker run -d --name ubuntu1 --network ssh-network ubuntu-ssh
docker run -d --name ubuntu2 --network ssh-network ubuntu-ssh
```

### Step 6: Configure SSH in Containers

To allow SSH connections between the containers, we need to know the IP addresses of the containers or use their names in the Docker network.

You can get the IP addresses of the containers using:

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu1
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu2
```

Alternatively, Docker resolves container names automatically in the same network, so you can SSH using the container names (e.g., `ubuntu2`).

### Step 7: SSH from One Container to Another

Now, log into one container and SSH into the other. Start by accessing one of the containers:

```bash
docker exec -it ubuntu1 bash
```

Once inside `ubuntu1`, attempt to SSH into `ubuntu2`:

```bash
ssh ubuntuuser@ubuntu2
```

You will be prompted for the password, which was set to `password` in the Dockerfile.

After entering the password, you should be able to log in to `ubuntu2` from `ubuntu1`.

## Verification

To verify the connection, you can try running commands on the second container once connected via SSH. For example, try:

```bash
hostname
```

This should return the hostname of the second container (`ubuntu2`), confirming the SSH connection is successful.

## Cleaning Up

Once you are done with the project, you can clean up the containers and network:

```bash
docker rm -f ubuntu1 ubuntu2
docker network rm ssh-network
```

You can also remove the image created:

```bash
docker rmi ubuntu-ssh
```

## Conclusion

In this project, we successfully set up and configured an SSH connection between two Ubuntu containers in Docker. This setup allows secure communication between containers using SSH, which is useful for containerized environments where inter-container communication is required.
