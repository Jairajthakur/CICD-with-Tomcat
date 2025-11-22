# 🚀 CICD with Tomcat

A professional **CI/CD pipeline project** integrating **Jenkins**, **Ansible**, and **Apache Tomcat** for automated Java application deployment. This setup enables smooth code integration, testing, and deployment with a single automated pipeline.

---

## 📘 Project Overview

This project demonstrates how to implement a **complete DevOps CI/CD workflow** for a Java web app using Jenkins and Ansible, deploying onto an Apache Tomcat server. It automates:

* Source code checkout from GitHub
* Build process (Maven/Gradle)
* Testing and code verification
* Application deployment via Ansible
* Environment configuration with Tomcat

---

## 📁 Folder Structure

```
CICD-with-Tomcat-main/
├── context.xml              # Tomcat context configuration
├── deploy.yml               # Ansible playbook for deployment
├── pipeline.groovy          # Jenkins scripted pipeline
├── tomcat-users.xml         # Tomcat users and roles
├── tomcat.yml               # Tomcat installation & setup playbook
└── README.md                # Project documentation
```

---

## ⚙️ Tools & Technologies

| Tool                  | Purpose                                        |
| --------------------- | ---------------------------------------------- |
| **Jenkins**           | Automates build, test, and deployment stages   |
| **Ansible**           | Configuration management and remote deployment |
| **Tomcat**            | Application server for hosting Java web apps   |
| **GitHub**            | Source code repository                         |
| **Maven/Gradle**      | Build automation tool                          |
| **Docker (Optional)** | Containerized Jenkins or Tomcat setup          |

---

## 🔁 CI/CD Pipeline Stages

1. **Checkout:** Jenkins pulls source code from GitHub.
2. **Build:** Maven/Gradle compiles the project and packages it into a WAR file.
3. **Test:** Automated tests are executed to validate the build.
4. **Deploy:** Jenkins triggers Ansible to deploy the WAR file to Tomcat.
5. **Verify:** The pipeline confirms deployment success and notifies via console or email.

---

## 🧩 Jenkins Setup

1. **Install Jenkins** and required plugins:

   * Git
   * Pipeline
   * Ansible
   * SSH Agent
2. **Add credentials** in Jenkins:

   * GitHub access token or SSH key
   * SSH key for remote deployment servers
3. **Create a new Pipeline job:**

   * Choose *Pipeline script from SCM*
   * Repository URL → Your GitHub project URL
   * Script path → `pipeline.groovy`

Once done, run the job to trigger the complete CI/CD process.

---

## 🧰 Ansible Deployment

To deploy manually via Ansible:

```bash
ansible-playbook -i inventory deploy.yml
```

### Example Inventory:

```
[webservers]
192.168.1.10 ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

## 🧱 Tomcat Configuration

Example entry in `tomcat-users.xml`:

```xml
<user username="deployer" password="StrongPassword123" roles="manager-script,admin-gui"/>
```

---

## 🔐 Security Practices

* Disable direct root SSH access.
* Use SSH keys instead of passwords.
* Restrict Jenkins and Tomcat access via firewall.
* Use role-based access in Jenkins.

---

## 🏗️ How to Run Locally

1. Clone the repository:

   ```bash
   git clone https://github.com/yourusername/CICD-with-Tomcat.git
   cd CICD-with-Tomcat-main
   ```
2. Configure Jenkins and Ansible as per setup steps.
3. Run Jenkins pipeline or Ansible playbook.

---

## 🏷️ Badges

![Jenkins](https://img.shields.io/badge/Jenkins-Automation-blue?logo=jenkins)
![Ansible](https://img.shields.io/badge/Ansible-Playbook-red?logo=ansible)
![Tomcat](https://img.shields.io/badge/Tomcat-Server-yellow?logo=apache)
![GitHub](https://img.shields.io/badge/Code-GitHub-black?logo=github)
![DevOps](https://img.shields.io/badge/CI%2FCD-Automation-success)

---

## 👨‍💻 Author

**Jairaj Singh**
*DevOps Engineer | Automating CI/CD pipelines using Jenkins, Ansible, and Tomcat*

👨‍💻 Author

📧 Email: th.jairaj@gmail.com</br>
🌐 GitHub: https://github.com/Jairajthakur</br>
💼 LinkedIn: https://linkedin.com/in/jairajsinghchauhan</br>


