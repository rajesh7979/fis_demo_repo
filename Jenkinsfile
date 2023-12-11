pipeline {
  agent any
  tools {
      maven 'maven3'
      jdk 'JDK8'
      jfrog 'cli'
    }
   environment {
        DOCKER_USERNAME = 'rajfriends1437@gmail.com'
	DOCKER_IMAGE_NAME = "$DOCKER_REGISTRY/$DOCKER_REPO/petclinic.${BUILD_ID}.${env.BUILD_NUMBER}"
        ARTIFACTORY_ACCESS_TOKEN = credentials('jf_access_token')
        WEBHOOK_URL = credentials("webhook_url")
        BUILD_NAME = "${JOB_NAME}"
        BUILD_NO = "${env.BUILD_NUMBER}"
	DOCKER_REGISTRY = 'slk.jfrog.io'
        DOCKER_REPO = 'docker-images-io-docker'
        DOCKER_PASSWORD = credentials('JFROG_PASSWORD')
	DOCKER_TAG = 'latest'
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
	stage('Push image to JFROG') {
	   steps {
	      sh "docker login -u $DOCKER_USERNAME -p '$DOCKER_PASSWORD' $DOCKER_REGISTRY"
	      sh "docker tag $DOCKER_IMAGE_NAME $DOCKER_IMAGE_NAME:$DOCKER_TAG"
              sh "docker push $DOCKER_IMAGE_NAME:$DOCKER_TAG"
			}
		}
        
}
}
