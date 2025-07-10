pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_USER = 'rishabhraj7'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Tag Docker Images') {
            steps {
                script {
                    def services = ['vote', 'result', 'worker', 'seed-data']
                    for (svc in services) {
                        bat "docker build -t %DOCKER_USER%/${svc}:%IMAGE_TAG% .\\${svc}"
                    }
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-creds', url: '']) {
                    script {
                        def services = ['vote', 'result', 'worker', 'seed-data']
                        for (svc in services) {
                            bat "docker push %DOCKER_USER%/${svc}:%IMAGE_TAG%"
                        }
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                    bat 'echo 🔄 Applying all Kubernetes manifests...'
                    bat 'kubectl apply -f k8s-specifications\\'

                    bat 'echo 🔄 Updating images for deployments...'
                    bat "kubectl set image deployment/vote vote=%DOCKER_USER%/vote:%IMAGE_TAG% -n %K8S_NAMESPACE%"
                    bat "kubectl set image deployment/result result=%DOCKER_USER%/result:%IMAGE_TAG% -n %K8S_NAMESPACE%"
                    bat "kubectl set image deployment/worker worker=%DOCKER_USER%/worker:%IMAGE_TAG% -n %K8S_NAMESPACE%"

                    bat 'echo 🔍 Checking rollout status...'
                    bat "kubectl rollout status deployment/vote -n %K8S_NAMESPACE% --timeout=60s"
                    bat "kubectl rollout status deployment/result -n %K8S_NAMESPACE% --timeout=60s"
                    bat "kubectl rollout status deployment/worker -n %K8S_NAMESPACE% --timeout=60s"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Voting App deployed successfully to AKS!'
        }
        failure {
            echo '❌ Deployment failed. Please check logs above.'
        }
    }
}
