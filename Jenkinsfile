pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        DOCKER_IMAGE = 'lmahdyyyy/student-management'   // your DockerHub repo
        K8S_DIR = 'k8s'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/lmahdy/Mahdi_Mzoughi_TWIN2.git'
            }
        }

        stage('Maven Build') {
            steps {
                withMaven(maven: 'M3') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_HUB_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh """
                        docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        sed 's|REPLACE_IMAGE|${DOCKER_IMAGE}:${BUILD_NUMBER}|g' ${K8S_DIR}/spring-deployment.yaml \
                        > /tmp/spring-deploy-${BUILD_NUMBER}.yaml

                        kubectl apply -f /tmp/spring-deploy-${BUILD_NUMBER}.yaml -n devops
                    """
                }
            }
        }

        stage('Restart App') {
            steps {
                sh "kubectl rollout restart deployment/spring-deployment -n devops"
            }
        }

        stage('Cleanup Local Images') {
            steps {
                sh "docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} || true"
                sh "docker rmi ${DOCKER_IMAGE}:latest || true"
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded! Application deployed to Kubernetes."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
