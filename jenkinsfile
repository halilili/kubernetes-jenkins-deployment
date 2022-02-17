pipeline {
    agent any
    
    tools {
      maven 'MavenBuild'
    }
    
    environment {
      DOCKER_TAG = getVersion()
    }
    
    stages {
        
        
        stage('SCM') {
            steps {
                git credentialsId: 'github-access', 
                    url: 'https://github.com/halilili/docker-ansible-jenkins-deployment'
                
            }
        }
        
        
        stage('Build Our Code') {
            steps {
               sh "mvn clean package"
            }
        }
        
        
        stage('Docker Build') {
            steps {
              sh "docker build . -t hassanali70826/myapp:${DOCKER_TAG}"
            }
        }
        
        
        stage('DockerHub Push') {
            steps {
                withCredentials([string(credentialsId: 'docker-password', variable: 'dockerHubPassword')]) {
                    sh "docker login -u hassanali70826 -p ${dockerHubPassword}"
                }
                
                sh "docker push hassanali70826/myapp:${DOCKER_TAG}"
            }
        }
        
        stage('Ansible Deployment and Packages Preperation') {
            steps {
                ansiblePlaybook credentialsId: 'aws-jenkins-server-aws-ssh-conn', disableHostKeyChecking: true, extras: " -e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'ansible-deployment.yml'
               
            }
        }
    }
}

def getVersion()
{
  def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
  return commitHash
}
