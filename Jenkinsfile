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
        dockerTool 'Docker --latest'
    }

    stages {

        stage('Tests & Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh "mvn clean package -DskipTests"
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        def backendImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", ".")
                        
                        docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                            backendImage.push("${DOCKER_TAG}")
                            backendImage.push("latest")
                        }
                    }
                }
            }
             post {
                success {
                    script {
                    def current = env.BUILD_NUMBER.toInteger()
                    
                    if (current > 1) {
                        def previousTag = (current - 1).toString()
                        
                        echo "Suppression de l'ancienne image : ${DOCKER_IMAGE}:${previousTag}"
                        sh "docker rmi ${DOCKER_IMAGE}:${previousTag} || true"
                    } 
                }
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
            echo "✅ Ca a chou - Image backend générée et poussée avec succès"
        }
        aborted {
            echo "🚫 Tué, tué, tué"
        }
        failure {
            echo "❌ Gaing gaing gaing - Échec des tests, du build ou du push"
        }
    }
}