pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        DOCKER_IMAGE = 'lmahdyyyy/student-management'
        // Nom du serveur SonarQube configuré dans Jenkins
        SONAR_SERVER = 'SonarQube Local'
    }
    stages {
        stage('Checkout & Build' ) {
            steps {
                // 1. Récupérer le code depuis GitHub
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/lmahdy/Mahdi_Mzoughi_TWIN2.git'
                
                // 2. Construire le projet (clean, test, package et génération du rapport JaCoCo )
                sh 'mvn clean verify'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                // Lancer l'analyse SonarQube
                withSonarQubeEnv(installationName: "${SONAR_SERVER}") {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=student-management -Dsonar.projectName=student-management'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    sh "docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                    sh "docker logout"
                }
            }
        }
        stage('Cleanup') {
            steps {
                sh "docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                sh "docker rmi ${DOCKER_IMAGE}:latest"
            }
        }
    }
    post {
        success {
            echo "Pipeline terminée avec succès ! Le livrable est dans target, l'image est sur Docker Hub et l'analyse SonarQube est disponible."
        }
        failure {
            echo "La pipeline a échoué."
        }
    }
}
