@Library('shared-library-jenkins') _
pipeline {
  agent any
 /* tools {
      maven 'maven3'
      jdk 'JDK8'
      jfrog 'cli'
    }*/
   environment {
        DOCKER_USERNAME = credentials('JFROG_USER')
	DOCKER_IMAGE_NAME = "$DOCKER_REGISTRY/$DOCKER_REPO/webapp.${env.BUILD_NUMBER}"
        ARTIFACTORY_ACCESS_TOKEN = credentials('jf_access_token')
        WEBHOOK_URL = credentials("webhook_url")
        BUILD_NO = "${env.BUILD_NUMBER}"
	DOCKER_REGISTRY = 'fisdemo1.jfrog.io'
        DOCKER_REPO = 'fis-demo-dockerhub-docker-local'
        DOCKER_PASSWORD = credentials('JFROG_PASSWORD')
	DOCKER_PASS = credentials('w_docker_pass')
	DOCKER_TAG = 'latest'
    }
    parameters {
        choice(name: 'action', choices:'create\ndelete', description: 'Choose create/destroy')
       // string(name: 'ImageName', description: 'name of the docker build', defaultValue: 'javapp')
       //  string(name: 'ImageTag', description: 'Tag of the docker build', defaultValue: 'v1')
       //  string(name: 'DockerHubUser', description: 'name of App', defaultValue: 'shaykube')
    }
	
    stages {
        stage('Git Checkout'){
        when { expression { params.action == 'create'} }
            steps{
                    gitCheckout(
                        branch: "main",
                        url: "https://github.com/rajesh7979/fis_demo_repo.git"
                    )             
            }     
        }     
        stage('Maven Build'){
        when { expression { params.action == 'create'} }

               steps {
                  script {
                    mvnBuild()
                  }
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
	stage('Push image to Dockerhub') {
	   steps {
	        withCredentials([usernameColonPassword(credentialsId: 'w_docker', variable: 'w_docker')]) {
                sh "docker login -u ahmedwahi314 -p $DOCKER_PASS"
	        sh "docker tag $DOCKER_IMAGE_NAME ahmedwahi314/petclinic.${BUILD_ID}:$DOCKER_TAG"
                sh "docker push ahmedwahi314/petclinic.${BUILD_ID}:$DOCKER_TAG"
            }
		}
	}
	/* stage('Publish build info') {
	   steps {
		jf 'rt build-publish'
		}
	} */
 
        stage('Notification') {
            steps {
                office365ConnectorSend webhookUrl: '$WEBHOOK_URL',
                message: 'build is success',
                status: 'Success'            
            }
        }
    }
	post {
          success {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '.', reportFiles: 'report.html', reportName: 'Trivy Scan', reportTitles: 'Trivy Scan', useWrapperFileDirectly: true])
            echo 'Build successfully!'   
        }

          failure {
            echo 'Build Failed'
        }
    }       
}
