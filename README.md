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

vi /etc/ansible/hosts</br>
## Ex 1: Ungrouped hosts, specify before any group headers.</br>
[Prod]</br>
172.31.20.40</br>
172.31.21.25</br>

Lets Generate SSH Keys, using this KEY Ansible server will communicate with worker nodes

ssh-keygen

ssh-copy-id root@private ip of prod-1</br>
ssh-copy-id root@private ip of prod-2</br>

# Step 3: Setup SonarQube

install sonar

==============
#! /bin/bash</br>
cd /opt/</br>
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip</br>
unzip sonarqube-8.9.6.50800.zip</br>
amazon-linux-extras install java-openjdk11 -y</br>
useradd sonar</br>
chown sonar:sonar sonarqube-8.9.6.50800 -R</br>
chmod 777 sonarqube-8.9.6.50800 -R</br>
su - sonar</br>
# use the below command manually after installation</br>
#sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start</br>
#echo "user=admin & password=admin"</br>
=================

sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start

http://IP:9000</br>
username: admin</br>
password: admin</br>

Add project --> Manually --> project=hotstarapp --> generate token--> hotstarapp --> click on maven</br>
31edb5e1b676bd8e6b4d8d78d4ea0ced41692375

Install Plugins
--------------
SonarQube Scanner</br>
Maven Integration plugins</br>
Sonar Scanner Quality Gates</br>
Nexus</br>
Ansible</br>
deploy to container</br>
S3 Publisher</br>

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


Now Store the Artifacts to S3
================================
🔹 Step 1: Install the Plugin

Go to Jenkins Dashboard → Manage Jenkins → Manage Plugins.

Search for "S3 publisher" and "Pipeline: AWS Steps"(optional) install it.

🔹 Step 2: Configure AWS Credentials</br>
Go to Manage Jenkins → System → Amazon S3 profiles --> ProfileName=s3creds --> access key and secret key - Save</br>

Go to your Jenkins Job → Configure -->  </br>
 --> Post-build Actions.</br>
 → Select "Publish artifacts to S3 bucket".</br>
 --> Source = **/*.war</br>
 --> Destination Bucket = jen-test-me-reyaz/</br>
 --> Bucket Region = ap-south-1</br>
 --> Server side Encryption</br>


Step 4 --> Launch ubuntu 24 EC2 for Nexus t2.medium

install Nexus
------------

## update the system packages</br>
sudo apt-get update</br>

## #1: Install OpenJDK 17 on Ubuntu 24.04 LTS</br>
apt install openjdk-17-jdk openjdk-17-jre -y</br>

##Download the SonaType Nexus on Ubuntu using wget</br>
cd /opt</br>
sudo wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz  </br>


##Extract the Nexus repository setup in /opt directory</br>
tar -zxvf latest-unix.tar.gz    </br>

##Rename the extracted Nexus setup folder to nexus</br>
sudo mv /opt/nexus-3.72.0-04 /opt/nexus</br>

##As security practice, not to run nexus service using root user, so lets create new user named nexus to run nexus service</br>
sudo adduser nexus   --> give password</br>


##To set no password for nexus user open the visudo file in ubuntu</br>
sudo visudo</br>

##Add below line into it , save and exit</br>
nexus ALL=(ALL) NOPASSWD: ALL</br>

##Give permission to nexus files and nexus directory to nexus user</br>
sudo chown -R nexus:nexus /opt/nexus</br>
sudo chown -R nexus:nexus /opt/sonatype-work</br>

##To run nexus as service at boot time, open /opt/nexus/bin/nexus.rc file, uncomment it and add nexus user as shown below</br>
sudo vi /opt/nexus/bin/nexus.rc</br>
run_as_user="nexus"  </br>


##To Increase the nexus JVM heap size, you can modify the size as shown below</br>
vi /opt/nexus/bin/nexus.vmoptions</br>

-Xms1024m</br>
-Xmx1024m</br>
-XX:MaxDirectMemorySize=1024m</br>

-XX:LogFile=./sonatype-work/nexus3/log/jvm.log</br>
-XX:-OmitStackTraceInFastThrow</br>
-Djava.net.preferIPv4Stack=true</br>
-Dkaraf.home=.</br>
-Dkaraf.base=.</br>
-Dkaraf.etc=etc/karaf</br>
-Djava.util.logging.config.file=/etc/karaf/java.util.logging.properties</br>
-Dkaraf.data=./sonatype-work/nexus3</br>
-Dkaraf.log=./sonatype-work/nexus3/log</br>
-Djava.io.tmpdir=./sonatype-work/nexus3/tmp</br>

##To run nexus as service using Systemd</br>
sudo vi /etc/systemd/system/nexus.service</br>


[Unit]
Description=nexus service</br>
After=network.target</br>

[Service]
Type=forking</br>
LimitNOFILE=65536</br>
ExecStart=/opt/nexus/bin/nexus start</br>
ExecStop=/opt/nexus/bin/nexus stop</br>
User=nexus</br>
Restart=on-abort</br>

[Install]
WantedBy=multi-user.target

sudo systemctl start nexus

sudo systemctl enable nexus

sudo systemctl status nexus  

tail -f /opt/sonatype-work/nexus3/log/nexus.log

ufw allow 8081/tcp   --> if required

http://server_IP:8081  

Creating Repo
=============

Login , username: admin, password: cat /opt/nexus/sonatype-work/nexus3/admin.password
Disable anonymous access

Click on Setting Symbol --> Repositories --> Create repository --> maven2(hosted) --> name(projectname = hotstarapp) --> save

Version Policy --> Snapshot --> Deployment policy --> allow to redeploy

In Jenkins --> Manage Jenkins --> Credentials --> Username and password =--> Username: admin, Password: reyaz123, ID = nexuscreds


Step 5 : Create a Jenkins pipeline
-----------------------------------

First install plugins - Nexus and Ansible, sonar, maven, deploy to container, S3 Publisher
---------------------

Jenkins --> Manage Jenkins --> plugins --> nexus artifact upload, SonarQube Scanner, Maven Integration, deploy to container and ansible --> install --> restart jenkins

Manage Jenkins --> tools --> Add ansible --> Name=ansible, path = /bin

pipeline code till artifacts and run the pipeline --> it will generate war file

pipeline {
    agent any
   
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/ReyazShaik/java-project-maven-new.git&#39;
            }
        }
        stage('build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('artifact') {
            steps {
                sh 'mvn package'
            }
        }
    }
}

**Build now***

Integrate with Sonar
-------------------

pipeline {
    agent any
   
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/ReyazShaik/java-project-maven-new.git&#39;
            }
        }
        stage('build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }

        stage('artifact') {
            steps {
                sh 'mvn package'
            }
        }
    }
}


**Build now***

Integrate with Nexus
--------------------
pipeline {
    agent any
   
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/ReyazShaik/java-project-maven-new.git&#39;
            }
        }
        stage('build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('artifact') {
            steps {
                sh 'mvn package'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'myapp', classifier: '', file: 'target/myapp.war', type: '.war']], credentialsId: 'nexuscreds', groupId: 'in.reyaz', nexusUrl: '13.233.124.23:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'hotstarapp', version: '8.3.3-SNAPSHOT'
            }
        }
    }
}

S3 Integration
=============

pipeline {
    agent any
   
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/ReyazShaik/java-project-maven-new.git&#39;
            }
        }
        stage('build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }

        stage('artifact') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Upload to S3') {
            steps {
                s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'jenkins-artifacts-demo-test', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: false, noUploadOnFailure: false, selectedRegion: 'ap-south-1', showDirectlyInBrowser: false, sourceFile: '**/*.war', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: true]], pluginFailureResultConstraint: 'FAILURE', profileName: 's3creds', userMetadata: []
            }
        }
    }
}





**Build now***

Now Deployment to Production Nodes using Ansible
--------------------------------------------------

Go to Ansible master Server

Install tomcat in all Production nodes using Ansible
---------------------------------------------------

vi tomcat.yml

---
- name: Setup Tomcat
  hosts: all
  become: yes
  tasks:
    - name: Download tomcat from dlcdn
      get_url:
        url: "https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.10/bin/apache-tomcat-11.0.10.tar.gz&quot;
        dest: "/root/"

    - name: untar the apache file
      command: tar -zxvf apache-tomcat-11.0.10.tar.gz

    - name: Rename the tomcat
      command: mv apache-tomcat-11.0.10 tomcat

    - name: Install the latest available Java (OpenJDK)
      yum:
        name: java-17-amazon-corretto
        state: present

    - name: Setting the roles in tomcat-users.xml file
      template:
        src: tomcat-users.xml
        dest: /root/tomcat/conf/tomcat-users.xml

    - name: Delete two lines in context.xml
      template:
        src: context.xml
        dest: /root/tomcat/webapps/manager/META-INF/context.xml
    - name: Create Tomcat systemd Service File
      copy:
        dest: /etc/systemd/system/tomcat.service
        content: |
          [Unit]
          Description=Apache Tomcat Server
          After=network.target

          [Service]
          User=root
          Group=root
          Type=forking
          Environment="JAVA_HOME=/usr/lib/jvm/jre"
          Environment="CATALINA_HOME=/root/tomcat"
          ExecStart=/root/tomcat/bin/startup.sh
          ExecStop=/root/tomcat/bin/shutdown.sh
          Restart=on-failure

          [Install]
          WantedBy=multi-user.target
    - name: Reload systemd
      systemd:
        daemon_reload: yes
    - name: Start tomcat Service
      service:
        name: tomcat
        state: started
        enabled: yes


================================================================================

sed -i 's/87/93/g' tomcat.yml  -- if required


--> instead changing tomcat-user.xml and context.xml in all nodes just create files locally and replace in worker nodes

--> vi tomcat-users.xml
   this file is in git-hub copy paste the content --it has all changed usernames etc
--> vi context.xml
   this file is in git-hub copy paste the content --it has all changed usernames etc

ansible-playbook tomcat.yml

run the playbook --> it will install tomcat in all worker nodes
check with workernodes ip, tomcat is working or not

Now artifacts are in /var/lib/Jenkins/workspace/project/targets

cd /var/lib/Jenkins/workspace/project/target

now create another playbook

Note: In below playbook, change the source to your target project directory

vi deploy.yml
---
- name: Deploy war to tomcat servers
  hosts: all
  tasks:
    - name: task1
      copy:
        src: /var/lib/jenkins/workspace/project/target/myapp.war
        dest: /root/tomcat/webapps

RUn the playbook

ansible-playbook deploy.yml

--------> this is manual work, not good, now use pipelines

Integrate ansible to Jenkins
===========================

Manage Jenkins --> TOOLS --> Ansible --> name = ansible, path = /bin

Manage Jenkins --> Credentials --> username and password --username = root , password = rooot123456 (password while setting up ssh connection to tomcat servers from ansible ssh) , ID = linuxcreds

or

Manage Jenkins --> Credentials --> SSH username with Private Key --username = ec2-user , id = linuxcreds
Private Key -> Enter Directly--> Add --> pem file key


mv deploy.yml /etc/ansible

cd /etc/ansible --> all ansible files are in this folder
   
open pipeline --> add deploy stage --> generate pipeline syntax -->

        stage('Run Ansible Playbook') {
            steps {
               
            }
 

Sample Step = ansibleplaybook:invoke an ansible playbook

Ansible tool : ansible  
Playbook file path in workspace = /etc/ansible/deploy.yml
Inventory file path in workspace =  /etc/ansible/hosts
SSH connection credentials = linuxcreds
disable ssh host key check --> check it --> rest all defaults

Below code will come
-------------------
ansiblePlaybook credentialsId: 'linuxcreds', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: '/etc/ansible/deploy.yml', vaultTmpPath: ''


Optional:
(host subset - we can give test or dev or prod , no need to give in playbook host: dev)
(in host subset give $server = prod)
copy the script and put in deploy section in pipeline --> in pipeline ansible stage  see limit : '$server' will come

Below code will come
-------------------
ansiblePlaybook credentialsId: 'tomcatcreds', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', limit: '$server = prod', playbook: '/etc/ansible/deploy.yml', vaultTmpPath: ''


before executing pipeline, undeploy application from tomcat by going to browser of workernodes

pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/ReyazShaik/java-project-maven-new.git&#39;
            }
        }
        stage('build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('artifact') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Run Ansible Playbook') {
            steps {
                ansiblePlaybook credentialsId: 'linuxcreds', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: '/etc/ansible/deploy.yml', vaultTmpPath: ''
            }
        }
    }
}



run the pipeline

update the code in GitHub --> and click on build now

How to add PARAMETERS - no need to give host subset use parameters , host : all in pipeline
------------------------------------------------------
The Project is parameterized
Name: server
choices parameter:  -- pass only single choice
dev
test
prod

if string parameter -- multiple choices if you want to pass
Name: server

save

run the build with parameters  -- for string parameters give dev, test

In below pipeline code changed to limit: '$server' , previously it was limit: '$server = prod'


                ansiblePlaybook credentialsId: 'tomcatcreds', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', limit: '$server', playbook: '/etc/ansible/deploy.yml', vaultTmpPath: ''



if you want to checkout from another branch , we can parameter the branch name
Add choice parameter: branch , choices : main, hotfix, ,master, etc

in pipeline script

            steps{
                git branch: '$branch' url: 'https://github.com/ReyazShaik/java-project-maven-new.git&#39;
            }
        }
