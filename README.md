# Continuous Integration and Continuous Deployment (CI/CD) Pipeline

This repository contains the configuration files and code to provision, configure, and automate the deployment of a static web application. The pipeline leverages Terraform for infrastructure provisioning, Ansible for configuration management, Jenkins for continuous integration, SonarQube for static code analysis, and Nginx as the final web server.

## Architecture Flow

1. **Provisioning (Terraform):** Creates two virtual machines in Azure. One VM is designated for the Nginx web server, and the other hosts the CI/CD tools (Jenkins and SonarQube).
2. **Configuration (Ansible):** Connects to the provisioned VMs to install dependencies, configure Nginx, and deploy the Dockerized environments for Jenkins and SonarQube.
3. **Continuous Integration (Jenkins):** Pulls the application code from the repository, executes static code analysis, and transfers the artifacts.
4. **Code Quality (SonarQube):** Analyzes the source code to ensure it meets predefined quality gates before deployment.
5. **Deployment:** Pushes the approved static files to the Nginx server.

## Infrastructure and Configuration Adjustments

To ensure a stable and functional environment, the following adjustments were made to the baseline configuration:

### Terraform Updates
* **OS Image Upgrade:** Updated the `source_image_reference` to a modern operating system release to support current Python versions required by Ansible.
* **Dynamic Resource Allocation:** Modified the `main.tf` file to dynamically assign a larger instance size for the CI/CD server using `size = each.value == "jenkins" ? "Standard_B2ms" : var.size_servers` to prevent Out-Of-Memory errors.
* **Inventory Management:** Populated an Ansible `inventory` file with the public IP outputs from Terraform to establish proper host routing.

### Docker and Environment Fixes
* **Database Compatibility:** Pinned the PostgreSQL container image to `postgres:12` to resolve initialization failures and version incompatibilities with SonarQube.
* **Jenkins Integration:** Created a `plugins.txt` file (including `git`, `workflow-aggregator`, `pipeline-stage-view`, `blueocean`, `sonar`, `ssh-slaves`, `credentials`, `pipeline-utility-steps`) and updated the Jenkins Docker image to ensure all necessary tools are installed upon initialization.
* **Container Network Tools:** Installed essential network utilities inside the running Jenkins container to allow the pipeline to download the SonarScanner and transfer files.

## Jenkins Pipeline Configuration

The following configurations were applied within the Jenkins UI to secure and route the pipeline tasks:

* **Internal SonarQube Routing:** Configured the SonarQube server connection using the internal Docker URL `http://sonarqube:9000` to ensure a reliable connection immune to public IP changes.
* **Deployment Credentials:** Added global credentials for the Nginx VM, mapping the `adminuser` to the required authentication method to allow Jenkins to deploy artifacts securely.

## Execution Guide

Execute the following commands in sequential order to replicate the environment and trigger the deployment.

**Step 1: Provision Infrastructure**
Navigate to the Terraform directory to initialize the provider and deploy the virtual machines.
```bash
terraform init
terraform plan
terraform apply -auto-approve
```

**Step 2: Configure Servers**
Update the Ansible `inventory/hosts.ini` file with the new public IPs provided by the Terraform output. Then, navigate to the Ansible directory and execute the configuration playbook.
```bash
ansible-playbook -i inventory/hosts.ini playbook.yml
```

**Step 3: Inject Container Dependencies**
Access the newly created Jenkins/SonarQube virtual machine via SSH. Inject the required networking and transfer tools directly into the running Jenkins container.
```bash
docker exec -u root jenkins bash -c "apt-get update && apt-get install -y wget unzip sshpass"
```

**Step 4: Execute Pipeline**
Access the Jenkins web interface on port 8080, link the GitHub repository using a Pipeline project type, and click "Build Now" to start the CI/CD process.

## Deployment Verification

Below is the evidence of the successful pipeline execution and application deployment.

### 1. Code Quality Analysis

<img width="1086" height="296" alt="image" src="https://github.com/user-attachments/assets/76f38e1d-cf7e-48c9-8704-6db36197c164" />


### 2. CI/CD Pipeline Execution

<img width="785" height="402" alt="image" src="https://github.com/user-attachments/assets/ffd695b3-6968-4767-99e9-09604574a4b2" />


### 3. Live Application

<img width="1515" height="861" alt="image" src="https://github.com/user-attachments/assets/36762530-4829-409d-b755-bc97a51f0faa" />
