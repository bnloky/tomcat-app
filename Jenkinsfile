pipeline {
    agent any
    tools {
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
	       
        /*stage("SonarQube Analysis"){
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

        }*/
	 stage('Build and Push Docker Image') {
       
             steps {
               script {
                    sh ' docker build -t ${IMAGE_NAME} .'
                    def dockerImage = docker.image("${IMAGE_NAME}")
                    docker.withRegistry('https://index.docker.io/v1/', "dockerhub") {
                    dockerImage.push("${IMAGE_TAG}")
	    
            }
        }
      }
    }
	 stage("Update the Deployment Tags") {
            steps {
                script {
                    // Navigate to the tomcat-app-cd directory
                    dir("${WORKSPACE}/tomcat-app-cd") {
                        // Display the content of deployment.yaml before modification
                        sh 'ls deployment.yaml'
                        
                        // Update the deployment.yaml file
                        sh "sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml"
                        
                        // Display the content of deployment.yaml after modification
                        sh 'cat deployment.yaml'
                    }
                }
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                script {
                    // Navigate to the tomcat-app-cd directory
                    dir("${WORKSPACE}/tomcat-app-cd") {
                        // Configure Git user information
                        sh 'git config --global user.name "Rojha-git"'
                        sh 'git config --global user.email "raj199.com@gmail.com"'

                        // Add and commit changes to deployment.yaml
                        sh 'git add deployment.yaml'
                        sh 'git commit -m "Updated Deployment Manifest"'

                        // Push changes to the remote repository on GitHub
                        withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                            sh 'git push https://Rojha-git:${GIT_PASSWORD}@github.com/Rojha-git/tomcat-app-cd main'
                        }
                    }
                }
            }
        }
          
    

   }   
}	
