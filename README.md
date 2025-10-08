# Capstone-Project

## Problem Statement:
Design and implement an end-to-end cloud infrastructure automation and deployment solution using Terraform, Ansible, Docker, and Kubernetes.

## Terraform Task:

### Problem Statement: 
Launch a Ubuntu EC2 instance (t2.micro) to be used as your terraform workstation. From that WS, using Terraform, launch an EC2 
instance (instance type: t2.micro, OS: Ubuntu) to be used as an ansible 
workstation for the ansible task. Please make sure that you create a key (using ssh-keygen) and use it while launching the EC2 so that we can SSH into the 
ansible WS once it is created.

#### Steps:
In your terraform WS, install terraform using following commands
```
sudo apt update
```
```
sudo apt install wget unzip -y
```
```
wget https://releases.hashicorp.com/terraform/1.10.3/terraform_1.10.3_linux_amd64.zip
```
```
unzip terraform_1.10.3_linux_amd64.zip
```
```
sudo mv terraform /usr/local/bin
```
```
rm terraform_1.10.3_linux_amd64.zip
```

Install the aws cli
```
sudo apt-get install python3-pip -y
```
```
sudo pip3 install awscli
```

Use aws configure and give your credentials
```
aws configure
```

Create a directory and inside that directory create your Terraform files to create an instance and ssh into it.
```
mkdir lab
```
```
cd lab
```
```
vi main.tf
```
Copy and post the below configuration
```
provider "aws" {
  profile = "default" # This line is not mandatory.
  region  = "us-east-1"
}
 
resource "aws_instance" "ec2" {
  instance_type = "t2.micro"
  ami = "ami-023c11a32b0207432"                                  
  key_name = "capstone-key"
  depends_on = [ aws_key_pair.capstone-key ]                      #The Key should be created first
  vpc_security_group_ids = [aws_security_group.terraform_sg.id]   #attaching a security group for ssh
  tags = {
    Name = "Ansible Server"}
}

#Generating the Key pair
resource "tls_private_key" "capstone_key_pair" {
  algorithm = "RSA"
  rsa_bits  = 4096
}
#Storing the Public key in AWS
resource "aws_key_pair" "capstone-key" {
  key_name   = "capstone-key"
  public_key = tls_private_key.capstone_key_pair.public_key_openssh  #Passing the Public Key
}
 
#Store the private Key on Local
resource "local_file" "mykey_private" {
  content = tls_private_key.capstone_key_pair.private_key_pem
  filename = "capstone-key"
}
resource "local_file" "mykey_public" {
  content = tls_private_key.capstone_key_pair.public_key_openssh
  filename = "capstone-key.pub"
}


#Creating the security Group and enabling port 22 for ssh
resource "aws_security_group" "terraform_sg" {
  name        = "capstone-allow-ssh"
  description = "security group that allows ssh and all egress traffic"
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "capstone-allow-ssh"
  }
}
```
Initialise the directory
```
terraform init
```
Plan
```
terraform plan
```
Apply
```
terraform apply -auto-approve
```
Once the resources are created, login into the newly created Instance using the below command
```
ssh -i "capstone-key" ubuntu@IP
```

## Ansible Tasks:

### Problem Statement: 

Once you have created a new instance using Terraform (as part of Terraform task), ssh into that instance and install Ansible in it. After that, you have to install httpd webserver in the managed node. You do not have separate managed nodes. So use your ansible workstation itself as the managed node by adding the below line in your host inventory file:
localhost ansible_connection = local

#### Steps:
Install ansible using the following commands
```
sudo apt update
```
```
sudo apt install python3 python3-pip wget -y
```
```
sudo pip3 install boto boto3 ansible
```
```
ansible --version
```

Create a inventory in the location /etc/ansible/hosts and add the below
```
localhost ansible_connection=local
```
Create a directory
```
mkdir ansible-lab && cd ansible-lab
```
```
vi playbook.yaml
```
```
- name: This play will install httpd web servers on all the hosts
  hosts: all
  become: yes
  tasks:
    - name: Task1 will install web-server
      apt:
        name: apache2
        update_cache: yes
        state: latest
    - name: Task2 will start the web-server
      service:
        name: apache2
        state: started
```
Execute the playbook
```
ansible-playbook playbook.yaml
```
Access the webserver on the Ip of the same machine on port 80
