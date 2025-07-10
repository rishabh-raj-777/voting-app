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
            withCredentials([file(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG')]) {
                bat 'echo 🔄 Applying all Kubernetes manifests...'
                bat 'kubectl apply -f k8s-specifications\\'

                bat 'echo 🔄 Updating images for deployments...'
                bat 'kubectl set image deployment/vote vote=rishabhraj7/vote:1 -n default'
                bat 'kubectl set image deployment/result result=rishabhraj7/result:1 -n default'
                bat 'kubectl set image deployment/worker worker=rishabhraj7/worker:1 -n default'

                bat 'echo 🔍 Checking rollout status...'
                bat 'kubectl rollout status deployment/vote -n default --timeout=60s'
                bat 'kubectl rollout status deployment/result -n default --timeout=60s'
                bat 'kubectl rollout status deployment/worker -n default --timeout=60s'
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
