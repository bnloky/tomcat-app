pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'java11'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "tomcat-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "rajf5"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Rojha-git/tomcat-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

        stage("Test Application"){
            steps {
                 sh "mvn test"
           }
	}
	       
        stage("SonarQube Analysis"){
            steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }
	stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }

         stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    def dockerImage = docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")

                    // Authenticate with Docker Hub using Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Log in to Docker Hub
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"

                        // Push the Docker image to Docker Hub
                        sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }
    

   }   
}	
