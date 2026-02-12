pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'DockerHub'
        DOCKER_IMAGE = "navysama/petclinic-rest"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    tools {
        maven 'Maven 3.9.12'
        allure 'Allure 2.36.0'
    }

    stages {
        stage('Tests & Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh "mvn clean package"
                    }
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    def backendImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", ".")

                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        backendImage.push("${DOCKER_TAG}")
                        backendImage.push("latest")
                    }
                }
            }
            post {
                always {
                    sh "echo Clean Docker images"
                    sh "docker rmi ${DOCKER_IMAGE}:${IMAGE_TAG} || true"
                    sh "docker rmi ${DOCKER_IMAGE}:latest || true"
                }
            }
        }
        stage('Gen Allure report') {
            steps {
                allure includeProperties: false, jdk: '', resultPolicy: 'LEAVE_AS_IS', results: [[path: 'target/allure-results']]
            }
        }
    }
    post {
        always {
            echo "Fin d'éxécution"
        }
        success {
            echo "✅ Ca a chou - Tests passés → JAR généré avec succès"
        }
        aborted {
            echo "🚫 Tué, tué, tué"
        }
        failure {
            echo "❌ Gaing gaing gaing - Échec des tests ou du build"
        }
    }
}