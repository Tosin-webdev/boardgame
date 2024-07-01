# Phase 1 | Infra Setup

Step 1: Launch EC2 instance
In this Project we will be Creating 7 EC2 instances. One to run kubernetes master node, two slave nodes. One to run jenkins pipeline for our CI/CD one for nexus repository, sonarqube and for monitoring 

1. Sign in to the AWS management console:

   * Go to the AWS Management Console
   * Sign in with your AWS account details
  
2. Navigate to EC2:
   * Search for "EC2" in the search bar
  
3. Launch Instance:
   * Click "instances" in the EC2 dashboard.
   * Click the "Launch Instance" button
  
4. Choose an Amazon Machine Image (AMI)
   * Select "Ubuntu" from the list of available AMIs
   * Choose "Ubuntu Server 24.04 LTS"
   * confirm changes

5. Choose an Instance type:
   * Select an instance type (e.g t2 micro for testing)
   * Click "Next: Configure Instance Details"

6. Create or Select a Key pair
   * Select keypair which will be used to log into the Ec2 instance

7. Create or Select existing security group

8. Configure storage
   For kubernetes
   * increase the root volume to 30 GB to ensure enough space to work on the application.
     
9. Review and Launch
    
10.  Access the instance
   Connect to the instance using SSH.

Step 2: CLone the Code

git clone https:...

step 3 Install Docker 
Set up Docker on the EC2 instance:
1. Install prerequsite packages:
sudo apt-get update
sudo apt-get install ca-certificates curl

2. Download and add Docker's official GPG key:
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

3. Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

4. Update package index
sudo apt-get update

5. Install docker packages
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin.

### step 4: Setting Up jenkins on ubuntu

1. Update the system
   sudo apt-get update
   sudo apt-get upgrade -y
2. Install java (Jenkins requires Java)
   sudo apt install openjdk-17-jre-headless

3. Add jenkins repository key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
4. Add Jenkins repository
 echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

5. Update the package index.
   sudo apt-get update
6. Install jenkins
    sudo apt-get install jenkins -y
7. Start and enable Jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
8. Access Jenkins:
   Open a web browser and go to http://your_server_ip:8080.

### Step 5 Installing Trivy on Jenkins server

1. Install prerequisite packages:
    sudo apt-get update
    sudo apt-get install -y wget apt-transport-https gnupg lsb-release

2.  Add Trivy GPG key
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

3. Add Trivy repository to APT sources
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
    | sudo tee /etc/apt/sources.list.d/trivy.list > /dev/null
   
4. Update APT package index
sudo apt-get update

5. Install Trivy
sudo apt-get install -y trivy

## Step 6 - Setting Up Nexus Repository Manager Using Docker
### Step-by-Step Installation
1. Pull the nexus Docker image
   sudo docker pull sonatype/nexus3
2. Run the Nexus container
   sudo docker run -d  --name Nexus -p 8081:8081 sonatype/nexus3
3. Access Nexus

## Step 7 - Setting Up Nexus Repository Manager Using Docker
1. Run PostgreSQL container
2. docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

## Phase 3 CICD Pipeline

Step 1 - Install neccessary plugins in jenkins 
Go to manage Jenkins --> Plugins --> Avaliable plugins -->
Install Below plugins
1. Eclipse termurin Installer
2. Sonarqube scanner
3. Docker
4. Docker pippeline
5. Email Extension plugin
6. maven
7. config file provider
8. pipeline maven integration
9. kubernetes
10. kubernetes credentials
11. kubernetes client API
12. kubernetes CLI

   Configure the installed plugins
Goto Manage Jenkins → Tools → Install JDK(17), maven(3), docker,SonarQube Scanner → Click on Apply and Save

Add credentials

1. Add docker hub credentials
   Go to "dashboard" --> "

## Jenkins pipeline script

Phase 4 | Monitoring

In this set you will set up prometheus, grafana, node-exporte, blackbox-exporter
## sterp 1.
1. Install prometheus
   wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
2. Extract prometheus
   tar -xzvf prometheus-2.52.0.linux-amd64.tar.gz
3. Move to the extracted directory
   cd prometheus-2.53.0.linus-amd-64
4. Run prometheus
   ./prometheus
5. Verify prometheus is running
   Open a web browser and navigate to http://localhost:9090

Install Node Exporter
1. Download Node expoerter:
   wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
2. Extract the Tarball
3. 
