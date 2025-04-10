pipeline {
    agent any

    environment {
        IMAGE_NAME = "akash6637/java-microservice"
        REGISTRY = "docker.io"
    }

    tools {
        maven 'maven3'    
        jdk 'jdk17'         
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            when {
                anyOf {
                    branch 'develop'
                    branch pattern: "feature/.*", comparator: "REGEXP"
                }
            }
            environment {
                SONARQUBE_SCANNER_PARAMS = "-Dsonar.projectKey=java-microservice"
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            when {
                branch 'develop'
            }
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'develop'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin $REGISTRY
                        docker push $IMAGE_NAME:$BUILD_NUMBER
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
