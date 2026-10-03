pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKER_USER = 'aymen177' // Remplacez par votre nom d'utilisateur Docker Hub
        IMAGE_NAME = "backend-app"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test Maven') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                // 1. Création de l'image Docker
                bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                // 2. Publication de l'image Docker (nécessite l'ID credentials 'docker-hub-credentials' dans Jenkins)
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER_ENV', passwordVariable: 'DOCKER_PASS_ENV')]) {
                    bat "docker login -u %DOCKER_USER_ENV% -p %DOCKER_PASS_ENV%"
                    bat "docker tag ${IMAGE_NAME}:${IMAGE_TAG} %DOCKER_USER_ENV%/${IMAGE_NAME}:${IMAGE_TAG}"
                    bat "docker push %DOCKER_USER_ENV%/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Déploiement MySQL') {
            steps {
                // 3. Lancement du conteneur MySQL s'il n'existe pas déjà
                bat '''
                docker stop mysql || exit 0
                docker rm mysql || exit 0
                docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=timesheet -p 3306:3306 mysql:8.0
                '''
            }
        }

        stage('Déploiement backend-app') {
            steps {
                // 4. Déploiement du backend (suppression de l'ancien + lancement du nouveau)
                bat '''
                docker stop backend-app || exit 0
                docker rm backend-app || exit 0
                docker run -d --name backend-app -p 8082:8080 --link mysql:mysql %DOCKER_USER%/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Vérification avec logs') {
            steps {
                // 5. Vérification des conteneurs et logs
                bat 'docker ps'
                bat 'docker logs backend-app'
            }
        }
    }
}