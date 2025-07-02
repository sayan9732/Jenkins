# Jenkins
Jenkins notes 2025

#################################
Build and deployment process
##################################

1. source code (take from git hub)
2. compilation
3. unit testing
4. package code as .jar file
5. create docker image
6. run docker image inside a container
----------------------------------


The above steps where performed by us manually in our docker classes. Question is how can we automate these tasks. Can we do these steps manually to deploy our application in different environments like dev, QA, staging, UAT, production etc?

#############
Jenkins CI/CD
#############

-> Jenkins is a popular open-source automation server used for Continuous Integration (CI) and Continuous Deployment (CD) in software development. It helps automate the building, testing, and deployment of applications.

-> Automation – Jenkins automates repetitive tasks like code compilation, testing, and deployment

-> Plugins – It has 1,800+ plugins, integrating with Git, Docker, Kubernetes, Maven, and more

-> Pipeline as Code – Uses Jenkinsfile (written in Groovy) to define CI/CD pipelines.

Jenkins CI/CD Workflow:
____________________________

-> Developers push code to a repository (GitHub, GitLab, Bitbucket).
-> Jenkins pulls the latest code and triggers a build using maven.
-> Artifact is generated (e.g., JAR, Docker image).
-> Deployment – Jenkins deploys the application to a test/staging/production environment
-> We can create jenkins pipeline in 2 ways

		1) Declarative Pipeline
		2) Scripted Pipeline (groovy)

Jenkins Declarative Pipeline Explained

A Jenkins Declarative Pipeline is a structured way to define a CI/CD pipeline using Groovy-based syntax. Let's break down your pipeline step by step.

Pipeline Structure:

pipeline {  
    agent any  // Runs the pipeline on any available agent (Jenkins node)

    stages {  // Defines different stages of the pipeline
        stage('git clone') {  // Stage 1: Clone the repository
            steps {  
                // Logic for cloning a Git repository
            }
        }
        stage('mvn build') {  // Stage 2: Build the project using Maven
            steps {  
                // Logic for running Maven build
            }
        }
        stage('build image') {  // Stage 3: Build a Docker image
            steps {  
                // Logic for building a Docker image
            }
        }
    }
}
Explanation of Each Section
1. pipeline {}
This is the root block that defines the entire CI/CD pipeline.

2. agent any
Specifies that the pipeline can run on any available Jenkins node (agent).

If you want to run it on a specific node, you can use:

groovy

agent { label 'my-node' }
3. stages {}
A stage represents a logical step in the pipeline.

Each stage contains steps {} where the actual actions are performed.


Jenkins Server Setup in Linux VM

Step - 1 : Create Linux VM in AWS

Create Ubuntu VM using AWS EC2 (t2.medium, because jenkins require minimum 4gb RAM)

Note: Enable 8080 Port Number in Security Group Inbound Rules


Step-2 : Install Java using below commands

sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version

Step-3 : To Install Jenkins use the below commands

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

Step-4 : Start Jenkins services using below commands

sudo systemctl enable jenkins
sudo systemctl start jenkins

Step-5 : Verify Jenkins setup

sudo systemctl status jenkins

Step-6 : Open jenkins server in browser using VM public ip of linux vm

http://public-ip:8080/

Step-7 : Copy jenkins admin pwd

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Step-8 : Create Admin Account & Install Required Plugins in Jenkins
____________________________________________________________________

Configure Maven as Global Tool in Jenkins
############################################

Manage Jenkins -> Tools -> Maven Installation -> Add maven
---------------------------------------------------------------

Setup Docker in Jenkins
##############################################

curl -fsSL get.docker.com | /bin/bash
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart jenkins
sudo docker version
-------------------------------------------------------------
pipeline example
Note: The sh command in a Jenkins Declarative Pipeline is used to execute shell (bash)
##############################################

pipeline {
    agent any
    
    tools {
        maven "maven-3.9.9"
    }

    stages {
        stage('Git Clone') {
            steps {
                give here git url
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}

--------------------------------------------------------------

First Jenkins JOB
_____________________________________________________________

pipeline {
    agent any
    
    tools {
        maven 'x-3.9.9'  
    }

    stages {
        stage('git clone') {
            steps {
                git branch: 'main', url: 'https://github.com/pankajmutha14/docker-test.git'
            }
        }
        stage('mvn') {
            steps{
                sh 'mvn clean test package'
            }
        }
         stage('build image') {
            steps{
                sh 'docker build -t psait/test1 .'
            }
        }
        stage('deployment'){
            steps{
                sh 'docker run -d -p 9090:8080 --name psait/test1'
            }
        }
    }
}

Example 2:
_______________________
pipeline {
    agent any
    
    tools {
        maven 'x-3.9.9'  
    }

    stages {
        stage('git clone') {
            steps {
                git branch: 'main', url: 'https://github.com/pankajmutha14/docker-test.git'
            }
        }
        stage('mvn') {
            steps{
                sh 'mvn clean test package'
            }
        }
         stage('build image') {
            steps{
                sh 'docker build -t psait/test1 .'
            }
        }
        stage('deployment'){
            steps{
                sh 'docker stop psait'
                sh 'docker rm psait'
                sh 'docker run -d -p 9090:8080 --name psait psait/test1'
            }
        }
    }
}

Example on Upstream and downstream in pipeline - 
--------------------------------------------------------
test-pipeline-ci
######################
pipeline {
    agent any
    
    tools {
        maven 'x-3.9.9'  
    }

    stages {
        stage('git clone') {
            steps {
                git branch: 'main', url: 'https://github.com/pankajmutha14/docker-test.git'
            }
        }
        stage('mvn') {
            steps{
                sh 'mvn clean test package'
            }
        }
         stage('build image') {
            steps{
                sh 'docker build -t psait/test1 .'
            }
        }
        
        stage('Trigger CD'){
            steps{
                build 'test-pipeline-CD'
            }
        }
        
    }
}

test-pipeline-CD
##################

pipeline {
    agent any

    stages {
        stage('Deployment') {
            steps {
                script {
                    sh '''
                    # Stop and remove the existing container if running
                    docker stop psait || true
                    docker rm psait || true

                    # Pull the latest image (optional)
                    docker pull psait/test1 || true

                    # Run the new container
                    docker run -d -p 9090:8080 --name psait psait/test1
                    '''
                }
            }
        }
    }
}
