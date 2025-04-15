pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-id')  // à définir dans Jenkins
        REGISTRY = "docker.io"
        DOCKER_USER = "${DOCKERHUB_CREDENTIALS_USR}"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Mmh-Chalal/Jenkins-datascientest.git', branch: "${env.BRANCH_NAME}"
            }
        }

        stage('Build & Push Images') {
            steps {
                script {
                    def services = ['movie-service', 'cast-service']
                    for (service in services) {
                        def imageName = "${DOCKER_USER}/${service}:${IMAGE_TAG}"
                        sh "docker build -t ${imageName} ${service}"
                        withDockerRegistry([credentialsId: 'dockerhub-id', url: ""]) {
                            sh "docker push ${imageName}"
                        }
                    }
                }
            }
        }

        stage('Deploy to Namespace') {
            when {
                anyOf {
                    branch 'dev'
                    branch 'qa'
                    branch 'staging'
                }
            }
            steps {
                script {
                    sh "kubectl apply -n ${env.BRANCH_NAME} -f k8s/${env.BRANCH_NAME}/"
                }
            }
        }

        stage('Manual Deploy to Prod') {
            when {
                branch 'master'
            }
            input {
                message 'Confirmer le déploiement en production ?'
            }
            steps {
                sh 'kubectl apply -n prod -f k8s/prod/'
            }
        }
    }
}
