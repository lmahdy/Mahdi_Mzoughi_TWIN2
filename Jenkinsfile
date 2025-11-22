pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        DOCKER_IMAGE = 'lmahdyyyy/student-management'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh "sudo docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "sudo docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    sh "sudo docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    sh "sudo docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
                    sh "sudo docker push ${DOCKER_IMAGE}:latest"
                    sh "sudo docker logout"
                }
            }
        }
        stage('Cleanup') {
            steps {
                sh "sudo docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                sh "sudo docker rmi ${DOCKER_IMAGE}:latest"
            }
        }
    }
    post {
        success {
            echo "Pipeline terminée avec succès ! Le livrable est dans target et l'image est sur Docker Hub."
        }
        failure {
            echo "La pipeline a échoué."
        }
    }
}
