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
                git url: 'https://github.com/Mmh-Chalal/Jenkins-datascientest.git', branch: "${env.BRANCH_NAME}"
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

        stage('Tests') {
            steps {
                script {
                    sh '''
                    echo "[TEST] Vérification de la présence des fichiers..."
                    test -f movie-service/app/main.py || (echo "[FAIL] main.py manquant dans movie-service" && exit 1)
                    test -f cast-service/app/main.py || (echo "[FAIL] main.py manquant dans cast-service" && exit 1)

                    echo "[TEST] Dockerfile check"
                    test -f movie-service/Dockerfile || exit 1
                    test -f cast-service/Dockerfile || exit 1

                    echo "[TEST] Vérification rapide des YAML"
                    find k8s/${BRANCH_NAME} -name "*.yaml" -exec grep apiVersion {} \\; || (echo "[FAIL] YAML suspects" && exit 1)

                    echo "[OK] Tous les tests ont réussi."
                    '''
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

        stage('Health Check') {
            when {
                anyOf {
                    branch 'dev'
                    branch 'qa'
                    branch 'staging'
                }
            }
            steps {
                script {
                    def check = sh(
                        script: """
                        echo "[CHECK] Vérification des pods dans ${env.BRANCH_NAME}..."
                        kubectl get pods -n ${env.BRANCH_NAME}
                        NOT_RUNNING=\$(kubectl get pods -n ${env.BRANCH_NAME} --no-headers | grep -v 'Running' | wc -l)
                        if [ "\$NOT_RUNNING" -ne 0 ]; then
                            echo "[FAIL] Des pods ne sont pas en Running"
                            exit 1
                        else
                            echo "[OK] Tous les pods sont en Running"
                        fi
                        """,
                        returnStatus: true
                    )
                    if (check != 0) {
                        error("Des pods ne sont pas en état Running.")
                    }
                }
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
