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
'''STEP-1: INSTALLING GIT JAVA-1.8.0 MAVEN</br>
- yum install git java-1.8.0-openjdk maven -y</br>'''

'''STEP-2: GETTING THE REPO (jenkins.io --> download -- > redhat)</br>
- sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo</br>
- sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key</br>'''

'''STEP-3: DOWNLOAD JAVA11 AND JENKINS</br>
- amazon-linux-extras install java-openjdk11 -y</br>
- yum install jenkins -y</br>
- update-alternatives --config java</br>'''

'''STEP-4: RESTARTING JENKINS (when we download service it will on stopped state)</br>
- systemctl start jenkins.service</br>
- systemctl status jenkins.service</br>'''

---

# Setup Ansible and their worker nodes

'''Install Ansible
- dnf install ansible2 -y</br>
- yum install python3 python3-pip -y</br>
- ansible --version</br>'''

# SETUP:
# Login to Jenkins+Ansible Server and setup connections to 2 Worker Nodes(tomcat1 and tomcat2)

## First set the password for root
'''- passwd root</br>
- set new password: jai123</br>'''

## Enable all server to login as root
'''- vi /etc/ssh/sshd_config (38 & 61 uncomment both lines)</br>
- systemctl restart sshd</br>
- systemctl status sshd</br>
- hostname -i</br>'''



