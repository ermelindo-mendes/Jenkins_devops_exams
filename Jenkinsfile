pipeline {
    agent any

    environment {
        DOCKER_USER = 'ermel78'
        
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
        KUBECONFIG = '/home/ubuntu/.kube/config'
        COMPOSE_PROJECT_NAME = 'app' 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Images') {
            steps {
                script {
                    sh "docker compose build movie_service cast_service"
                    sh "docker tag app-movie_service:latest ${DOCKER_USER}/movie-service:${IMAGE_TAG}"
                    sh "docker tag app-cast_service:latest ${DOCKER_USER}/cast-service:${IMAGE_TAG}"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_ID')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER_ID --password-stdin"
                        sh "docker push ${DOCKER_USER}/movie-service:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_USER}/cast-service:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                sh "helm upgrade --install movie-service ./charts --namespace dev --set image.repository=${DOCKER_USER}/movie-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=movie-service"
                sh "helm upgrade --install cast-service ./charts --namespace dev --set image.repository=${DOCKER_USER}/cast-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=cast-service"
            }
        }

        stage('Deploy to QA & Staging') {
            parallel {
                stage('QA') {
                    steps {
                        sh "helm upgrade --install movie-service ./charts --namespace qa --set image.repository=${DOCKER_USER}/movie-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=movie-service"
                        sh "helm upgrade --install cast-service ./charts --namespace qa --set image.repository=${DOCKER_USER}/cast-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=cast-service"
                    }
                }
                stage('Staging') {
                    steps {
                        sh "helm upgrade --install movie-service ./charts --namespace staging --set image.repository=${DOCKER_USER}/movie-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=movie-service"
                        sh "helm upgrade --install cast-service ./charts --namespace staging --set image.repository=${DOCKER_USER}/cast-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=cast-service"
                    }
                }
            }
        }

        stage('Deploy to PROD') {
            when {
                branch 'master' 
            }
            steps {
                input message: "Voulez-vous déployer en PRODUCTION ?", ok: "Oui, déployer !"
                
                sh "helm upgrade --install movie-service ./charts --namespace prod --set image.repository=${DOCKER_USER}/movie-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=movie-service"
                sh "helm upgrade --install cast-service ./charts --namespace prod --set image.repository=${DOCKER_USER}/cast-service --set image.tag=${IMAGE_TAG} --set fullnameOverride=cast-service"
            }
        }
    }
}