# CICD-with-Tomcat
## Integrating CICD Pipeline with Tomcat servers


# Architecture diagram
**Developers push code --> Jenkins(Ansible) --> deploy --> Nodes(tomcats)**

**checkout --> Build --> Test --> Artifacts --> Nexus/S3 --> Deploy(Tomcat)**

[Step 1](#Step-1) --> Push code to GitHub
[Step 2](#Step-2) --> Generate Artifacts
[Step 3](#Step-3) --> Storing the Artifacts
[Step 4](#Step-4) --> Installing Tomcat
[Step 5](#Step-5) --> Deploy the artifacts to nodes using Ansible

---

# Step 1 --> Keep the code in GitHub (use java-project-new)

**https://github.com/Jairajthakur/java-project-maven-new.git**

---

# Step 2 -->  Install Jenkins and Ansible in server using script

Install Jenkins
