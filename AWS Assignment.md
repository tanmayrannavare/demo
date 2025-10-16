# AWS Assignment
## 🔑 Prerequisites:
Laptop with internet access.
AWS Free Tier account (credit/debit card required for signup).
Basic Linux commands knowledge (yum, apt, curl).
### In AWS:
  VPC (10.0.0.0/16):SUBNET(PUBLIC(10.0.1.0/24) AND PRIVATE(10.0.2.0/24))
  ROUTE,NAT,SECURITY GROUP(SSH,HTTP AND PORT 8080 for docker) 
  Create  the key(pem) same for all ec2 virtual machine !
  
## Objectives
 Stand up a basic two-subnet VPC in AWS (public + private).
## Launch:
 Private EC2 (no public IP) — Ansible control node.
 use elastic ip for private ec2 for installatiion
 Jump host in public subnet — for SSH into the private EC2.
 ```
 if you dont know how to copy directly dont worry just view your downloaded pem file using notepad and copy the key and paste is to virtual machine using nano key.pem
 give permission to your key.pem to
 chmod 400 key.pem !!!
 ```
 Web server in public subnet.
 ```
  use sudo for docker command
 ``` 
 From the private EC2, run the provided Ansible playbook to install Apache on the public web server
 and serve a page with Name + Roll No.
 Additionally, containerize a simple Apache site by building a basic Docker image that serves the same page.
 ## Deliverables (brief)
 Public web server URL showing Name + Roll No.
 One screenshot of Ansible run completion.
 One screenshot of your Docker container serving the same page locally (docker run …).
 One diagram or bullet list of your VPC + subnets + instances (very brief).

## Installation
### Installing Ansible (on the Private EC2)
 Once your private EC2 instance is reachable (via SSH from your jump host), install Ansible using the package manager for your OS:

🟢 For Amazon Linux 2 / RHEL / CentOS
```
sudo yum update -y
sudo yum install -y ansible
```
or (if yum doesn’t have it)
```
sudo amazon-linux-extras install ansible2 -y
ansible --version
```
🔵 For Ubuntu / Debian
```
sudo apt update -y
sudo apt install -y ansible
ansible --version
```
## inventory.ini
```
[web]
<public-web-server-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/yourpath/key.pem
#for Amazon Linux 2
<public-web-server-ip> ansible_user=ec2-user ansible_ssh_private_key_file=/yourpath/key.pem
```
### Site.yml
```
---
- name: Bootstrap Apache and deploy Name/Roll page
  hosts: web
  become: true
  gather_facts: true

  vars:
    apache_pkg: "{{ 'httpd' if ansible_facts.os_family in ['RedHat','Amazon'] else 'apache2' }}"
    apache_svc: "{{ 'httpd' if ansible_facts.os_family in ['RedHat','Amazon'] else 'apache2' }}"
    doc_root: "/var/www/html"
    student_name: "{{ lookup('env','STUDENT_NAME') | default('Your Name', true) }}"
    roll_no: "{{ lookup('env','ROLL_NO') | default('0000', true) }}"

  tasks:
    - name: Ensure package cache is up to date (Debian/Ubuntu)
      apt:
        update_cache: yes
      when: ansible_facts.os_family == 'Debian'

    - name: Install Apache
      package:
        name: "{{ apache_pkg }}"
        state: present

    - name: Ensure Apache service is enabled and running
      service:
        name: "{{ apache_svc }}"
        state: started
        enabled: true

    - name: Deploy index.html with Name and Roll No
      copy:
        dest: "{{ doc_root }}/index.html"
        mode: '0644'
        content: |
          <!doctype html>
          <html>
            <head><meta charset="utf-8"><title>Student</title></head>
            <body style="font-family: system-ui; margin: 3rem;">
              <h1>{{ student_name }}</h1>
              <h2>Roll No: {{ roll_no }}</h2>
              <p>Deployed via Ansible ✅</p>
            </body>
          </html>

```
### Export your name and roll number
export STUDENT_NAME="xyz xyz"

export ROLL_NO="123456"

### run
```
ansible -i inventory.ini web -m ping
ansible-playbook -i inventory.ini site.yml
```
## Dockerfile (minimal Apache image)
Builds a tiny image that serves the same page. No external files needed.
```
# Simple Apache image that serves Name + Roll No
FROM httpd:2.4-alpine

ARG STUDENT_NAME="Your Name"
ARG ROLL_NO="0000"

# Create a minimal index.html at container build time
RUN printf '<!doctype html><html><head><meta charset="utf-8"><title>Student</title></head><body style="font-family: system-ui; margin: 3rem;"><h1>%s</h1><h2>Roll No: %s</h2><p>Served from Docker ✅</p></body></html>' \
    "$STUDENT_NAME" "$ROLL_NO" \
    > /usr/local/apache2/htdocs/index.html

EXPOSE 80
CMD ["httpd-foreground"]
```
Step 1: Update system
```
sudo apt update
sudo apt upgrade -y
```
Explanation: Updates package lists and upgrades existing packages.

Step 2: Remove old versions
```
sudo apt remove docker docker-engine docker.io containerd runc
```
Explanation: Ensures no legacy Docker is present.

Step 3: Install required packages
```
sudo apt install -y ca-certificates curl gnupg lsb-release
```
Explanation: These tools allow adding secure repositories and managing keys.

Step 4: Add Docker’s official GPG key
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
Explanation: Adds Docker’s signing key to verify package integrity.

Step 5: Set up repository
```
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Explanation: Adds Docker’s official repo to apt sources.

Step 6: Install Docker CE
```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
Explanation: Installs Docker engine, CLI, container runtime, and Compose plugin.

Step 7: Start and enable Docker service
```
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```
Explanation: Starts Docker now and on future boots.
## build docker file
```
docker build -t Your_File_Name .
```
-t student-apache tags your image with the name student-apache.
. tells Docker to look for the Dockerfile in the current directory.
## Run Docker Container
```
docker run -d -p 8080:80 --name my-student-web Your_File_Name
```
```
docker ps
```
-d: detached mode
-p 8080:80: map port 8080 on your host to port 80 in the container
--name: give your container a name
## Now visit http://localhost:8080 to see the deployed page.
## Docker Hub Account Creation
In your desktop go to following address:

https://hub.docker.com/
Click on Sign Up
Select Personal Tab
Enter your email address
Put your desired Username
Enter your password
Click on Sign Up

Step 4: Verify on Docker Hub
```
Go to https://hub.docker.com
Login to your account
Navigate to Repositories and confirm your image flask-demo-app:v1 is listed !!!
```
============================================================================================
## 📤 Push Docker Image to Docker Hub
After building and running your Docker image locally, follow the steps below to push it to Docker Hub.

🔧 Step 1: Tag Your Docker Image
```
docker tag Your_Image_Name Your_DockerHub_Username/Your_Repository_Name:latest
```
Your_Image_Name: the name of the image you built locally (e.g., student-apache)

Your_DockerHub_Username: your Docker Hub username

Your_Repository_Name: the name of your Docker Hub repository

🔐 Step 2: Log in to Docker Hub
```
docker login
```
Enter your Docker Hub username and password when prompted.

☁️ Step 3: Push the Image to Docker Hub
```
docker push Your_DockerHub_Username/Your_Repository_Name:latest
```
This uploads your image to your Docker Hub account.

✅ Step 4: Pull & Run the Image Anywhere
```
docker pull Your_DockerHub_Username/Your_Repository_Name:latest
docker run -d -p 8080:80 Your_DockerHub_Username/Your_Repository_Name:latest
```
## If not Worked
Step 1: Login to Docker Hub
```
docker login -u <your_username>
```
Enter your Docker Hub username and password when prompted.

Step 2: Tag Your Docker Image
```
Replace <your_dockerhub_username> with your actual Docker Hub username:
docker tag flask-demo-app <your_dockerhub_username>/flask-demo-app:v1
```
Step 3: Push the Image to Docker Hub
```
docker push <your_dockerhub_username>/flask-demo-app:v1
```
Pulls the image from Docker Hub
## Runs it and maps it to http://localhost:8080



