# Prerequisites
- **JDK**: 17 or 21  
- **Maven**: 3.9  

# Pipeline Configuration

### ✅ Plugins Required
- Git Integration  
- Maven Integration  
- Nexus Artifact Uploader  
- SonarQube Scanner  
- Build Timestamp Plugin  
- Slack Notification Plugin  
- Pipeline: GitHub  
- Pipeline: Shared Libraries  

### ✅ Tools in Jenkins
- **JDK 17**
- **Maven 3.9.x**
- **SonarQube Scanner 4.7.0.2747**

### ✅ Credentials in Jenkins
| ID          | Purpose                         |
|-------------|---------------------------------|
| sonartoken  | SonarQube authentication token  |
| slacktoken  | Slack bot/user token           |
| gitlogin    | GitHub/Bitbucket credentials   |
| nexuslogin  | Nexus Repository credentials   |

### ✅ Other Required Setup
- Git Webhook (push → Jenkins)
- SonarQube Webhook (quality gate → Jenkins)
- SonarQube Quality Gate defined
- Slack Workspace integration
- Nexus (Release, Snapshot, Group repos)
- Jenkins Shared Library repository

---

# Slack Configuration: 
Email:      devopspractice@myyahoo.com
Workspace:  Accenture (accenture-3hn2465)
Channel:    devops_practices
Token:      sLxuMSHJ3uCrisWYGPZPFyow
---

# 📦 CI/CD Pipeline — Jenkins + Shared Library + Nexus + SonarQube + Ansible

This repository contains the full CI/CD automation used for the **vProfile Application**, powered by:

- ✅ Jenkins Declarative Pipeline  
- ✅ Jenkins Shared Library  
- ✅ SonarQube Code Analysis  
- ✅ Nexus Repository Manager  
- ✅ Ansible (Blue-Green Deployments)  
- ✅ Slack Notifications  
- ✅ AWS EC2 Infrastructure  

---

## 🚀 Pipeline Overview

The CI/CD pipeline executes the following stages:

### ✅ 1. Environment Preparation
- Installs Ansible on the Jenkins build node
- Verifies installed tools:
  - Java
  - Maven
  - Git
  - Ansible

### ✅ 2. Build & Test
- Maven build (`mvn -DskipTests install`)
- Unit tests (`mvn test`)
- Checkstyle code-quality validation
- WAR file is archived in Jenkins

### ✅ 3. Static Code Analysis
- SonarQube Scanner executed using Shared Library
- Quality Gate enforced (pipeline stops on FAIL)

### ✅ 4. Artifact Management (Nexus)
- Auto-generates version (`BUILD_ID + timestamp`)
- Uploads the WAR file to Nexus Release repository

### ✅ 5. Deployment Workflow (Blue/Green)
- **Staging deployment (automated)**
- **Manual approval gate**
- **Production deployment**

Both deployments use Ansible playbooks with environment-specific inventory.

### ✅ 6. Notifications
- Slack notifications sent on:
  - SUCCESS
  - FAILURE
  - UNSTABLE
  - ABORTED
