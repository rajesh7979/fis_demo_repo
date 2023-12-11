pipeline {
  agent any
  tools {
      maven 'maven3'
      jdk 'JDK8'
      jfrog 'cli'
    }
   environment {
      DOCKER_IMAGE_NAME = "slk.jfrog.io/fis-demo-dockerhub/app-image.${BUILD_ID}.${env.BUILD_NUMBER}"
      ARTIFACTORY_ACCESS_TOKEN = credentials('jf_access_token')
      WEBHOOK_URL = credentials("webhook_url")
      BUILD_NAME = "${JOB_NAME}"
      BUILD_NO = "${env.BUILD_NUMBER}"
       
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
                 def customImage = docker.build("$DOCKER_IMAGE_NAME", "./docker")
                 }                     
           }
        }
	stage('Trivy Scan') {
            steps {
               sh 'trivy image $DOCKER_IMAGE_NAME  --output report.html || true'
	  }
        }
	stage('Scan and push image') {
	   steps {
	       dir('${WORKSPACE}') {
		// Scan Docker image for vulnerabilities
		//	jf 'docker scan $DOCKER_IMAGE_NAME'

		// Push image to Artifactory
		jf 'docker push $DOCKER_IMAGE_NAME'
				}
			}
		}
        
}
}
