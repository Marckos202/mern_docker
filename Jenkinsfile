pipeline {
    agent any 

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
        DOCKERHUB_USERNAME = 'marckos202'      
        APP_VERSION = "latest"
        BACKEND_IMAGE_NAME = "${DOCKERHUB_USERNAME}/mern-backend"
        FRONTEND_IMAGE_NAME = "${DOCKERHUB_USERNAME}/mern-frontend"
        K8S_FILES_PATH = '.' 
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo "Código fuente extraído."
            }
        }

        stage('Build Backend Image') {
            steps {
                    sh "docker build -t ${env.BACKEND_IMAGE_NAME}:${env.APP_VERSION} -f Dockerfile-backend ."
            }
        }

        stage('Build Frontend Image') {
            steps {
                    sh "docker build -t ${env.FRONTEND_IMAGE_NAME}:${env.APP_VERSION} -f Dockerfile-frontend ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS_ID, passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh "echo \$DOCKERHUB_PASSWORD | docker login -u \$DOCKERHUB_USER --password-stdin"
                }
            }
        }

        stage('Push Backend Image') {
            steps {
                sh "docker push ${env.BACKEND_IMAGE_NAME}:${env.APP_VERSION}"
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh "docker push ${env.FRONTEND_IMAGE_NAME}:${env.APP_VERSION}"
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                sh "sed -i 's|image: mern-backend:latest|image: ${env.BACKEND_IMAGE_NAME}:${env.APP_VERSION}|g' ${env.K8S_FILES_PATH}/backend-deployment.yaml"
                sh "sed -i 's|imagePullPolicy: Never|imagePullPolicy: Always|g' ${env.K8S_FILES_PATH}/backend-deployment.yaml"

                sh "sed -i 's|image: mern-frontend:latest|image: ${env.FRONTEND_IMAGE_NAME}:${env.APP_VERSION}|g' ${env.K8S_FILES_PATH}/frontend-deployment.yaml"
                sh "sed -i 's|imagePullPolicy: Never|imagePullPolicy: Always|g' ${env.K8S_FILES_PATH}/frontend-deployment.yaml"
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh "kubectl apply -f ${env.K8S_FILES_PATH}/mongo-deployment.yaml"
                    sh "kubectl apply -f ${env.K8S_FILES_PATH}/mongo-service.yaml"
                    sh "kubectl apply -f ${env.K8S_FILES_PATH}/backend-deployment.yaml"
                    sh "kubectl apply -f ${env.K8S_FILES_PATH}/backend-service.yaml"
                    sh "kubectl apply -f ${env.K8S_FILES_PATH}/frontend-deployment.yaml"
                    sh "kubectl apply -f ${env.K8S_FILES_PATH}/frontend-service.yaml"
                    sh "kubectl apply -f ${env.K8S_FILES_PATH}/ingress.yaml"

                    sh "echo 'Esperando a que los despliegues estén listos...'"
                    sh "kubectl rollout status deployment/mongo-deployment --timeout=180s || echo 'Mongo deployment timed out'"
                    sh "kubectl rollout status deployment/backend-deployment --timeout=180s || echo 'Backend deployment timed out'"
                    sh "kubectl rollout status deployment/frontend-deployment --timeout=180s || echo 'Frontend deployment timed out'"
                    sh "echo 'Despliegues completados (o timeout).'"
                }
            }
        }
    }

    post {
        always {
            script {
                sh 'docker logout'
            }
            echo 'Pipeline finalizado.'
        }
        success {
            echo '¡Pipeline ejecutado exitosamente!'
        }
        failure {
            echo 'El Pipeline falló.'
        }
    }
}