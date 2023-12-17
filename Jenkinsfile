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

        stage("Build & Push Docker Image") {
           steps {
               script {
                // Configure Docker
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                       docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_CONFIG') {
                       docker_image = docker.build "${IMAGE_NAME}"
                }
            }

            // Push Docker Image
            withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_CONFIG') {
                    docker_image.push("${IMAGE_TAG}")
                    docker_image.push('latest')
                }
            }
        }
    }
}
    
    
       

   }   
}	
