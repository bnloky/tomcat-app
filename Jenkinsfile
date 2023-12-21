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
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM (tomcat-app)") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Rojha-git/tomcat-app'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        // Uncomment the following stages if SonarQube integration is needed
        /*
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        */

        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                    def dockerImage = docker.image("${IMAGE_NAME}")
                    docker.withRegistry('https://index.docker.io/v1/', "dockerhub") {
                        dockerImage.push("${IMAGE_TAG}")
                    }
                }
            }
        }
        
        
        
        stage("Checkout from SCM (tomcat-app-cd)") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Rojha-git/tomcat-app-cd'
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    cat deployment.yaml
                    sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                    cat deployment.yaml
                """
            }
        }
        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                    git config --global user.name "Rojha-git"
                    git config --global user.email "raj199.com@gmail.com"
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                    
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/Rojha-git/tomcat-app-cd main"
                }
            }
        }
    }
}
