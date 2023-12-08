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
	stage('Checkstyle Analysis'){
            steps {
                    sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=petclinic \
                   -Dsonar.projectName=petclinic \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
		   -Dsonar.java.binaries=target/test-classes/org/springframework/samples/petclinic/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }
        stage('Build docker image') {
           steps {
               script {         
                 def customImage = docker.build('ahmedwahi314/petclinic', "./docker")
                 docker.withRegistry('http://20.65.200.234:8082/artifactory/dockerhub', 'jfrog') {
                 customImage.push("${env.BUILD_NUMBER}")
                 }                     
           }
        }
	  }
    }
}
