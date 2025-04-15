pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-id')
        DOCKER_USER = "${DOCKERHUB_CREDENTIALS_USR}"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Mmh-Chalal/Jenkins-datascientest.git', branch: 'dev'
            }
        }

        stage('Build & Push Images') {
            steps {
                script {
                    def services = ['movie-service', 'cast-service']
                    for (service in services) {
                        def image = "${DOCKER_USER}/${service}:${IMAGE_TAG}"
                        sh "docker build -t ${image} ${service}"
                        withDockerRegistry([credentialsId: 'dockerhub-id', url: '']) {
                            sh "docker push ${image}"
                        }
                    }
                }
            }
        }

        stage('Deploy to Dev/QA/Staging') {
            when {
                anyOf {
                    branch 'dev'
                    branch 'qa'
                    branch 'staging'
                }
            }
            steps {
                sh "kubectl apply -n ${env.BRANCH_NAME} -f k8s/${env.BRANCH_NAME}/"
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            input {
                message "Déployer en production ?"
            }
            steps {
                sh "kubectl apply -n prod -f k8s/prod/"
            }
        }
    }
}
