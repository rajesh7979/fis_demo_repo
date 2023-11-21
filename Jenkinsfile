pipeline {
  agent any
    tools {
      maven 'maven3'
                 jdk 'JDK8'
    }
    stages {      
        stage('Build maven ') {
            steps { 
                    sh 'pwd'      
                    sh 'mvn  clean install package'
            }
        }
        
        stage('Copy Artifact') {
           steps { 
                   sh 'pwd'
		   sh 'cp -r target/*.jar docker'
           }
        }
         
        stage('Build docker image') {
           steps {
               script {         
                 def customImage = docker.build('ahmedwahi314/petclinic', "./docker")
                 docker.withRegistry('https://ahmed314.azurecr.io', 'acr') {
                 customImage.push("${env.BUILD_NUMBER}")
                 }                    
           }
        }
	  }
    }
}
