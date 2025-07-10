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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat """
                        echo 🔁 Applying all Kubernetes manifests...
                        kubectl apply -f k8s\\

                        echo 🔁 Updating images for deployments...
                        kubectl set image deployment/vote-deployment vote=%DOCKER_USER%/vote:%IMAGE_TAG% -n %K8S_NAMESPACE%
                        kubectl set image deployment/result-deployment result=%DOCKER_USER%/result:%IMAGE_TAG% -n %K8S_NAMESPACE%
                        kubectl set image deployment/worker-deployment worker=%DOCKER_USER%/worker:%IMAGE_TAG% -n %K8S_NAMESPACE%

                        echo 🔍 Checking rollout status...
                        kubectl rollout status deployment/vote-deployment -n %K8S_NAMESPACE% --timeout=60s
                        kubectl rollout status deployment/result-deployment -n %K8S_NAMESPACE% --timeout=60s
                        kubectl rollout status deployment/worker-deployment -n %K8S_NAMESPACE% --timeout=60s
                    """
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
