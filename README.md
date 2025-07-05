# DevSecOps-jenkins-sonarqube-nexus
Building a Scalable Jenkins Pipeline Integrated with SonarQube and Nexus for Version-Aware Artifact Management

#### Overview of this project
 - CI/CD Pipeline Setup: Integrated Jenkins, SonarQube, and Nexus for a seamless build and deployment pipeline and automated code quality checks using SonarQube and artifact management using Nexus.
 - Building Robust Pipelines: Automatically deploy code from GitHub to Jenkins, analyze it with SonarQube, and push the resulting artifacts to Nexus for version control.
 - Maven Integration: Managed artifact versions (SNAPSHOT vs. RELEASE) to avoid overwriting and ensure version discipline using Nexus.
 - SonarQube Setup: Configured SonarQube for static code analysis and enforced quality gates before production and ensured only quality-verified code reaches production environments.
 - Deploy to Nexus: Configure Nexus for storing development and production artifacts, maintaining traceability and version control. Pushed SNAPSHOT and RELEASE artifacts to different repositories for better management.
 - Practical Demo: demo of setting up Jenkins, configuring SonarQube for analysis, and deploying artifacts to Nexus. Walkthrough on version management, quality control, and deployment automation.

#### Problem Statement
In modern DevOps-driven environments, delivering high-quality, version-controlled software rapidly is crucial. Manual build, test, and release processes often lead to inconsistencies, security risks, and delayed deployments. To overcome this, organizations implement Continuous Integration and Continuous Delivery (CI/CD) pipelines that ensure automated builds, code quality checks, and artifact management.
 - Automated code quality analysis
 - Version-controlled artifact management
 - Seamless integration of developer workflows
 - Enforcement of proper version tagging for artifact traceability

#### Why need this
In the modern DevOps landscape, Continuous Integration and Continuous Delivery (CI/CD) are at the heart of delivering high-quality software faster. A properly configured CI/CD pipeline ensures.
 - Rapid feedback through code analysis
 - Automated build and testing
 - Reliable artifact publishing for version control

#### When need this
 - Your team is building Java-based applications with Maven.
 - There’s a need for a centralized code quality gate before releasing artifacts.
 - You want to version-control all builds, either as production releases or development snapshots.
 - Your organization is adopting DevSecOps practices and wants traceability in artifact flow.
 - You’re deploying across multiple environments (e.g., QA, UAT, Prod) and need consistent artifacts from a central repository (Nexus).

### Architecture Overview

## Step-by-Step Implementation 

### Prerequisites
3 Ubuntu servers (Ubuntu 24.04) each with: 2 CPUs, 8 GB RAM minimum. Open required ports: Jenkins (8080), SonarQube (9000), Nexus (8081). Internet access on all servers.

#### Jenkins server
```
sudo su
sudo apt update

sudo apt install openjdk-17-jre-headless -y
sudo apt install maven -y

java --version
mvn -v

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key


echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null


sudo apt-get update

sudo apt-get install jenkins -y

jenkins --version
```
![1](images/01.png)
#### Sonarqube server

```
sudo apt update
sudo apt install openjdk-17-jre-headless -y

java --version

sudo apt install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo apt install docker-compose -y


sudo usermod -aG docker ${USER}

sudo systemctl enable docker

sudo systemctl restart docker
```


```
# Create docker volumes

# Create docker volumes:

docker volume create --name sonarqube_data
docker volume create --name sonarqube_logs
docker volume create --name sonarqube_extensions

docker volume list
```


```
# run sonarqube as container

docker run -d --name sonarqube \
    -p 9000:9000 \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    -v sonarqube_logs:/opt/sonarqube/logs \
    sonarqube:lts-community


# verify
docker ps
```
![2](images/2.png)
```
username - admin
password - admin
```
![3](images/3.png)
#### Nexus server
Nexus as a service on the server.
```
# Need to nexus as a separate user, so create a nexus user
sudo apt update
sudo su
```
```
adduser nexus
```
```
cd /opt
mkdir sonatype
chown -R nexus:nexus sonatype
cd sonatype

# download nexus tarball and extract
wget https://download.sonatype.com/nexus/3/nexus-3.81.1-01-linux-x86_64.tar.gz

tar xvzf nexus-3.81.1-01-linux-x86_64.tar.gz

# Change the ownership of the nexus files so we can run nexus service as user nexus
chown -R nexus:nexus *

mv nexus-3.81.1-01 nexus
```
```
# Create a nexus service
cd /etc/systemd/system/

vi nexus.service


# add the below content to the above nexus.service file

[Unit]
Description=nexus service
After=network.target
  
[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/sonatype/nexus/bin/nexus start
ExecStop=/opt/sonatype/nexus/bin/nexus stop

User=nexus
Restart=on-abort
TimeoutSec=600
  
[Install]
WantedBy=multi-user.target
```
```
# Reload the daemon, enable and start the service

sudo systemctl daemon-reload
sudo systemctl enable nexus.service
sudo systemctl start nexus.service

sudo systemctl status nexus.service
```

![4](images/4.png)

![5](images/5.png)

#### Get the maven-releases and maven-snapshots path
edit the porm.xml file in the github repositary 
![6](images/6.png)
Download below plugins in jenkins:  
 - SonarQube Scanner
 - Maven Integration
 - Config File Provider
 - Pipeline Maven Integration
 - Pipeline: Stage View

#### Create credentials [ Need to be ready with sonarqube & nexus ]
Credentials: 
 - sonar-token (use the token we created in sonarqube server)
 - maven-releases (nexus user id and password) - admin /12345
 - maven-snapshots (nexus user id and password) - admin /12345

![7](images/7.png)

![8](images/8.png)

#### SonarQube Scanner setting in jenkins
#### SonarQube Server setting in jenkins
