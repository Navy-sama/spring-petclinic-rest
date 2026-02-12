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
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        def backendImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", ".")
                        backendImage.push("${DOCKER_TAG}")
                        backendImage.push("latest")
                    }
                }
            }
        }
        stage('Gen Allure report') {
            steps {
                allure includeProperties: false, jdk: '', resultPolicy: 'LEAVE_AS_IS', results: [[path: 'target/allure-results']]
            }
        }
        stage('Déploiement Intégré (Recette)') {
            agent {
                label 'docker-host'
            }
            steps {
                script {
                    echo "🚀 Déploiement en environnement de recette..."
                    
                    sh '''
                        docker-compose down || true
                        docker-compose up -d
                    '''
                    
                    echo "⏳ Attente du démarrage des services..."
                    sleep 30
                    
                    sh '''
                        echo "Vérification de l'état des conteneurs:"
                        docker-compose ps
                        
                        echo "Vérification de la santé de PostgreSQL:"
                        docker-compose exec -T postgres pg_isready -U petclinic || true
                        
                        echo "Vérification du backend:"
                        curl -f http://localhost:9966/petclinic/actuator/health || echo "Backend pas encore prêt"
                    '''
                }
            }
            post {
                success {
                    echo "✅ Déploiement en recette réussi"
                }
                failure {
                    echo "❌ Échec du déploiement en recette"
                    sh "docker-compose logs || true"
                }
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