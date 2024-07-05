# Phase 1 | Infra Setup

## Step 1: Launch EC2 instance
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

## Install KUBECTL On Jenkins Server


```
vi k.sh
```
paste the command below into the file

```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
change file permission to be executable 
```
sudo chmod +x k.sh
```
run and install
```
./k.sh
```


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

```
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Tosin-webdev/boardgame'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=.'''
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }          
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t blackcypher01/boardgame:latest .'
                }
            }
        }
        stage('Docker Image Scan') {
            steps {
               sh 'trivy image --format table -o trivy-image-report.html blackcypher01/boardgame:latest'
            }
        }
        stage('Push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh 'docker push blackcypher01/boardgame:latest'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.59.153:6443') {
                    sh 'kubectl apply -f deployment-service.yaml'
                }
            }
        }
        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.59.153:6443') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
                def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
                """
                
                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'oladejit3@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                )
            }
        }
    }
}

```
## Kubernetes Setup
Create service account
```
vi svc.yaml
```
paste the command below into the file
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps

```
```
kubectl create ns webapps
```
```
kubectl apply -f svc.yaml
```
```
vi role.yaml

```
paste the command below into the file
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
- apiGroups:
  - ""  # Core API group
  - apps
  - autoscaling
  - batch
  - extensions
  - policy
  - rbac.authorization.k8s.io
  resources:
  - pods
  - componentstatuses
  - configmaps
  - daemonsets
  - deployments
  - events
  - endpoints
  - horizontalpodautoscalers
  - ingress
  - jobs
  - limitranges
  - namespaces
  - nodes
  - pods  # Duplicate entry, remove if not necessary
  - persistentvolumes
  - persistentvolumeclaims
  - resourcequotas
  - replicasets
  - replicationcontrollers
  - serviceaccounts
  - services
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - patch
  - delete
```
```
kubectl apply -f role.yaml
```
```
kubectl apply -f bind.yaml
```

Bind role to service account
```
vi bind.yaml
```
paste the content below into the file

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
- namespace: webapps
  kind: ServiceAccount
  name: jenkins

```
save and run the command
kubectl appply -f bind.yaml

Now the user we created have permision to perform deployment.

## create token for authentication

```
vi sec.yaml
```
paste the command below into the file
```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: jenkins
```

provide name of namespace
kubectl apply -f sec.yaml -n webapps


kubectl apply -f svc.yaml


provide name of namespace
Phase 4 | Monitoring

In this set you will set up prometheus, grafana, node-exporte, blackbox-exporter
## sterp 1.
1. Install prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
```
2. Extract prometheus
```
tar -xzvf prometheus-2.52.0.linux-amd64.tar.gz
```
3. Move to the extracted directory
```
cd prometheus-2.53.0.linus-amd-64
```
4. Run prometheus
```
./prometheus &
```
5. Verify prometheus is running
   Open a web browser and navigate to `http://your_server_ip::9090`

Install Node Exporter
1. Download Node expoerter:
```
   wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
```
2. Extract the Tarball
   
```
tar -xzvf node_exporter-1.8.1.linux-amd64.tar.gz
```
3. move to the extracted directory
```
cd node_exporter-1.8.1.linux-amd64
```
4. Run Node Exporter
```
  ./node_exporter
```
5. Verify Node Exporter
Open a web browser and navigate to http://your_server_ip:9100/metrics

## Step 3 - Install Blackbox exporter
1. Download blackbox exporter
   wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz

2. Extract the tarball
tar -xzvf blackbox_exporter-0.25.0.linux-amd64.tar.gz

3. Move to the Extracted Directory
   cd blackbox_exporter-0.25.0.linux-amd64
4. Run Blackbox exporter
   ./blackbox_exporter &
5. Verify Blackbox Exporter is running
   Open a web browser and navigate to http://localhost:9115/metrics.


## Step 4 - Install and Setup grafana

1. Install Dependencies
   * Update Package list
   * sudo apt-get update
2. Download nd Install grafana
   * Download the grafana engterprise package
```
wget https://dl.grafana.com/enterprise/release/grafana-
enterprise_11.0.0_amd64.deb
```
   * Install grafana using dpkg
```
sudo dpkg -i grafana-enterprise_11.0.0_amd64.deb
```
 
3. Start and Enable Grafana
   * Start the Grafana service:
   sudo system start grafan-server
   * Enable the Grafana service to start on boot:
      sudo systemctl enable grafana-server
     
4. Access grafana
   * Open web browser and navaigate to http://your_server_ip:3000
   * Log into grafana
       * The default username is admin
       * The default password is admin
   * Change the default password
     Upon first login, you will be prompted to change the default password. Enter
a new password and confirm it.

5. Configure grafana

 * Add a data Source
   * Navigate to configurations > Data sources
   * Click Add data source
   * Choose your desires data source type (prometheus)
   * Configure the data source with the appropriate URL (e.g Your server URL)
   * CLick save &test

 * Create a dashboard
   * Navigate to create > dashboard.
   * Add panels and configure quesroies to visualize metrics
   * save the dashboard
  
 ## Step 5 - Import a dashboard
 To make it easier to view metrics, you can import a pre configured dashboard. FOllow these steps:
   * Clik on the + plus icon in the left sidebar to open the "create" menu
   * Select "Dashboard"
   * Click on the "Import" dashboard option
   * Enter the code you want to import (e.g code 1860))
   * click the "load" button
   * Select the data source you added (prometheus) from the dropdown
   * Click on the import button
You should now have grafana dashnoard set up to visualize metrics from prometheus.
![Screenshot from 2024-06-29 12-36-41](https://github.com/Tosin-webdev/boardgame/assets/64624808/c6c06967-de11-4e00-a249-361f41731243)



![Screenshot from 2024-06-29 20-44-49](https://github.com/Tosin-webdev/boardgame/assets/64624808/19bf3b30-458b-4e53-ad14-3c3a44e96501)

