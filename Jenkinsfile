pipeline{
    agent any
    tools {
      maven 'maven'
    }
    environment {
      DOCKER_TAG = getVersion()
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/geelabalakrishna/dockeransiblejenkins.git'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t gbalakrishna/dockerjenkins:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u gbalakrishna -p ${dockerHubPwd}"
                }
                
                sh "docker push gbalakrishna/dockerjenkins:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'ansible-id', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
