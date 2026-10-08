pipeline {
    agent any

    environment {
        DOCKER_IMAGE = '24mis0114/student-event-registration'
    }

    stages {

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:v1 .'
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
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat 'docker push %DOCKER_IMAGE%:v1'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
            }
        }

        stage('Verify Replicas') {
            steps {
                bat 'kubectl get pods'
                bat 'kubectl get deployment'
                bat 'kubectl get service'
            }
        }

        stage('Update Application') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:v2 .'
                bat 'docker push %DOCKER_IMAGE%:v2'
                bat 'kubectl set image deployment/student-event-deployment student-event-container=%DOCKER_IMAGE%:v2'
            }
        }

        stage('Verify Updated Application') {
            steps {
                bat 'kubectl rollout status deployment/student-event-deployment'
                bat 'kubectl get pods'
                bat 'kubectl get service'
            }
        }
    }
}