pipeline {
    agent any

    environment {
        DOCKER_IMAGE = '24mis0114/student-event-registration'
        DOCKER_EXE = 'C:\\Users\\varsh\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Users\\varsh\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                bat "\"%DOCKER_EXE%\" build -t %DOCKER_IMAGE%:v1 ."
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat "\"%DOCKER_EXE%\" login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "\"%DOCKER_EXE%\" push %DOCKER_IMAGE%:v1"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat "\"%KUBECTL_EXE%\" apply -f deployment.yaml"
            }
        }

        stage('Verify Replicas') {
            steps {
                bat "\"%KUBECTL_EXE%\" get pods"
                bat "\"%KUBECTL_EXE%\" get deployment"
                bat "\"%KUBECTL_EXE%\" get service"
            }
        }

        stage('Update Application') {
            steps {
                bat "\"%DOCKER_EXE%\" build -t %DOCKER_IMAGE%:v2 ."

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat "\"%DOCKER_EXE%\" login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "\"%DOCKER_EXE%\" push %DOCKER_IMAGE%:v2"
                }

                bat "\"%KUBECTL_EXE%\" set image deployment/student-event-deployment student-event-container=%DOCKER_IMAGE%:v2"
            }
        }

        stage('Verify Updated Application') {
            steps {
                bat "\"%KUBECTL_EXE%\" rollout status deployment/student-event-deployment"
                bat "\"%KUBECTL_EXE%\" get pods"
                bat "\"%KUBECTL_EXE%\" get service"
            }
        }
    }
}