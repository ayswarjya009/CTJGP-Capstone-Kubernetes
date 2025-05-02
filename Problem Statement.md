# Capstone-Project

## Problem Statement:
Design and implement an end-to-end cloud infrastructure automation and deployment solution using Terraform, Ansible, Docker, and Kubernetes.

## Terraform Task:

### Problem Statement: 
Launch a Ubuntu EC2 instance (t2.micro) to be used as your terraform workstation. From that WS, using Terraform, launch an EC2 
instance (instance type: t2.micro, OS: Ubuntu) to be used as an ansible 
workstation for the ansible task. Please make sure that you create a key (using ssh-keygen) and use it while launching the EC2 so that we can SSH into the 
ansible WS once it is created.


## Ansible Tasks:

### Problem Statement: 

Once you have created a new instance using Terraform (as part of Terraform task), ssh into that instance and install Ansible in it. After that, you have to install httpd webserver in the managed node. You do not have separate managed nodes. So use your ansible workstation itself as the managed node by adding the below line in your host inventory file:
localhost ansible_connection = local

## Docker & Kubernetes Task:

### Problem Statement: 
1. Build a docker image to use the python api and push it to the DockerHub. 
Create a pod and nodeport service with that Docker image.
2. Two Tier Architecture: Deploy Nextcloud with PostgreSQL on Kubernetes
