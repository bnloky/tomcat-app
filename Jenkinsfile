pipeline {
    agent any
    tools {
        maven 'Maven3'
        jdk 'jdk17'
    }
    environment {
        APP_NAME = "tomcat-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "loky353"
        DOCKER_PASS = 'Lokesh@353'
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
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/bnloky/tomcat-app'
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
        
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
    

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

        stage('Trivy scan'){
            steps {
                script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image rajf5/tomcat-app-pipeline:${IMAGE_TAG} --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }
        stage('Cleanup Artifact'){
            steps { 
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage("Checkout from SCM (tomcat-app-cd)") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/bnloky/tomcat-app-cd'
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
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                sh """
                    git config --global user.name "bnlokesh"
                    git config --global user.email "lokeshnarasimha353@gmail.com"
                    sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                    git add deployment.yaml
                    git commit -m "Updated Deployment Manifest"
                    git push https://github.com/bnloky/tomcat-app-cd HEAD:main
                    
                """
                 
                }
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'lokeshnarasimha353@gmail.com',                              
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt' 
     }

 }

}


