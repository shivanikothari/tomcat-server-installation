pipeline {
    agent { label 'master' }
    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout()
    }

 stages {
        stage ('Checkout') {
            agent { 
                node {
                      label 'tomcat_nodes'
                      customWorkspace "ansible"
                     }
            }
            steps {
                script {
                     checkout scm
                }
            }
       }     
     
        stage ('InstallTomcat') {
            agent { 
                node {
                      label 'tomcat_nodes'
                      customWorkspace "ansible"
                      }
                  }
            steps {
                   ansiblePlaybook(
                   credentialsId: 'tomcat-nodes-ssh-key', 
                   inventory: 'inventory/hosts', 
                   playbook: 'plays/tomcat-installation.yml',                  
                   hostKeyChecking: false,
                   extras: '-e version=9.0.1'
                   )
                  }       
               }
    }	
}
