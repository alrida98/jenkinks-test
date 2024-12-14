pipeline {
    agent any 
    stages {
      stage('SCM Checkout') {
        steps {
           checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/alrida98/jenkinks-test.git']]])  
        }
      }  
      stage('Execute Ansible') {
        steps {
           ansiblePlaybook credentialsId: 'ubuntu-slaves-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'hosts', playbook: 'web-app.yml'
        }  
      }
    }
}
