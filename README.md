# CICD-with-Tomcat
## Integrating CICD Pipeline with Tomcat servers


# Architecture diagram
**Developers push code --> Jenkins(Ansible) --> deploy --> Nodes(tomcats)**

**checkout --> Build --> Test --> Artifacts --> Nexus/S3 --> Deploy(Tomcat)**

[Step 1](#Step-1----Keep-the-code-in-GitHub-use-java-project-new) --> Push code to GitHub</br>
[Step 2](#Step-2----Install-Jenkins-and-Ansible-in-server-using-script) --> Generate Artifacts</br>
[Step 3](#Step-3) --> Storing the Artifacts</br>
[Step 4](#Step-4) --> Installing Tomcatv</br>
[Step 5](#Step-5) --> Deploy the artifacts to nodes using Ansible</br>

---

# Step 1 --> Keep the code in GitHub (use java-project-new)

**https://github.com/Jairajthakur/java-project-maven-new.git**

---

# Step 2 --> Install Jenkins and Ansible in server using script

Install Jenkins
=====================================================
STEP-1: INSTALLING GIT JAVA-1.8.0 MAVEN</br>
- yum install git java-1.8.0-openjdk maven -y</br>

STEP-2: GETTING THE REPO (jenkins.io --> download -- > redhat)</br>
- sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo</br>
- sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key</br>

STEP-3: DOWNLOAD JAVA11 AND JENKINS</br>
- amazon-linux-extras install java-openjdk11 -y</br>
- yum install jenkins -y</br>
- update-alternatives --config java</br>

STEP-4: RESTARTING JENKINS (when we download service it will on stopped state)</br>
- systemctl start jenkins.service</br>
- systemctl status jenkins.service</br> 

---

# Setup Ansible and their worker nodes

Install Ansible
- dnf install ansible2 -y</br>
- yum install python3 python3-pip -y</br>
- ansible --version</br>

# SETUP:
# Login to Jenkins+Ansible Server and setup connections to 2 Worker Nodes(tomcat1 and tomcat2)

## First set the password for root
- passwd root</br>
- set new password: jai123</br>

## Enable all server to login as root
- vi /etc/ssh/sshd_config (38 & 61 uncomment both lines)</br>
- systemctl restart sshd</br>
- systemctl status sshd</br>
- hostname -i</br>

# Add Inventory file
=============

vi /etc/ansible/hosts
## Ex 1: Ungrouped hosts, specify before any group headers.
[Prod]
172.31.20.40
172.31.21.25

Lets Generate SSH Keys, using this KEY Ansible server will communicate with worker nodes

ssh-keygen

ssh-copy-id root@private ip of prod-1
ssh-copy-id root@private ip of prod-2

# Step 3: Setup SonarQube

install sonar

==============
#! /bin/bash
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
amazon-linux-extras install java-openjdk11 -y
useradd sonar
chown sonar:sonar sonarqube-8.9.6.50800 -R
chmod 777 sonarqube-8.9.6.50800 -R
su - sonar
# use the below command manually after installation
#sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start
#echo "user=admin & password=admin"
=================

sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start

http://IP:9000
username: admin
password: admin

Add project --> Manually --> project=hotstarapp --> generate token--> hotstarapp --> click on maven
31edb5e1b676bd8e6b4d8d78d4ea0ced41692375

Install Plugins
--------------
SonarQube Scanner,
Maven Integration plugins
Sonar Scanner Quality Gates
Nexus
Ansible
deploy to container,
S3 Publisher

RESTART JENKINS

Go to Credentials --> Kind= Secret text , secret = 45ac8e27ed329f054b9138db8573e883e44e8101 , id = sonar
------------------

Go to System --> SonarQube servers --> Enable Environment Variables -->
-----------
     Name = SonarQube
     URL = http://3.110.190.217:9000
     Server authentication token = select token


Go to  Tools --> SonarQube Scanner installations , Name = sonarscanner
------------

