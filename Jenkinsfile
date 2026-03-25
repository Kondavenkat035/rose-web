pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('valaxy-docker') // stored in Jenkins
        DOCKER_IMAGE = "kondavenkat035/rose-web:latest"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kondavenkat035/rose-web.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push Docker Image') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $DOCKER_IMAGE'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yml --validate=false'
                sh 'kubectl apply -f k8s/service.yml'
                sh 'kubectl apply -f k8s/ingress.yml'
            }
        }
    }
}
