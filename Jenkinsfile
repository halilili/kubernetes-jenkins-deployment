pipeline {
    agent any
    
    tools {
      maven 'MavenBuild'
    }
    
    environment {
      docker_tag = latestVersion()
    }
    
    
  stages{
      
      stage ('SCM Clone')
        {
            steps{
               //When we dont have master branch, we should mention the main branch, it is created newly
               git branch: 'main', credentialsId: 'github-access', url: 'https://github.com/halilili/kubernetes-jenkins-deployment'
               
            }
        }
        
        
        stage ('Maven Clean Package')
        {
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('SonarQube Analysis') {
            steps{
                echo "SonarQube Analysis Stage"
            }
        }
      
      stage('Jmeter Test') {
            steps{
                sh "/home/ubuntu/jmeter/apache-jmeter-5.4.3/bin/jmeter.sh -n -t /home/ubuntu/jmeter/apache-jmeter-5.4.3/bin/examples/CSVSample.jmx -l test.jtl"
            }
        }
    
        
        
       stage('Docker Build'){
           steps{
               sh "docker build . -t hassanali70826/my-app:${docker_tag}"
           }
       }
       
       stage('DockerHub Push'){
           steps{
               withCredentials([string(credentialsId: 'docker-password', variable: 'dockerHubPassword')]) {
                    sh "docker login -u hassanali70826 -p ${dockerHubPassword}"
               }
              
               sh "docker push hassanali70826/my-app:${docker_tag}"
           }
       }
       
       
       stage("Prepare Deployment Tag"){
           steps{
               sh "chmod +x changeTag.sh"
               sh "./changeTag.sh ${docker_tag}"
           }
       }
       
       
       stage('Deployment using jenkins kubernetes plugin') {
           steps {
               withCredentials([
                   string(credentialsId: 'aws-minikube-token', variable: 'api_token')
                   ]) {
                    sh 'kubectl --token $api_token --server https://192.168.49.2:8443 --insecure-skip-tls-verify=true apply -f k8s/deployment-manifest.yml'
               }
            }
       }
       
  }
  
}

def latestVersion(){
    def commitVersion = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitVersion
}

