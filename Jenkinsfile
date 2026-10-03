pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKER_USER = 'aymen177'
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
                bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER_ENV', passwordVariable: 'DOCKER_PASS_ENV')]) {
                    bat "docker login -u %DOCKER_USER_ENV% -p %DOCKER_PASS_ENV%"
                    bat "docker tag ${IMAGE_NAME}:${IMAGE_TAG} %DOCKER_USER_ENV%/${IMAGE_NAME}:${IMAGE_TAG}"
                    bat "docker push %DOCKER_USER_ENV%/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Déploiement MySQL') {
            steps {
                bat 'docker stop mysql 2>NUL || ver >NUL'
                bat 'docker rm mysql 2>NUL || ver >NUL'
                bat 'docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=timesheet -p 3306:3306 mysql:8.0'
            }
        }

        stage('Déploiement backend-app') {
            steps {
                bat 'docker stop backend-app 2>NUL || ver >NUL'
                bat 'docker rm backend-app 2>NUL || ver >NUL'
                bat "docker run -d --name backend-app -p 8082:8080 --link mysql:mysql %DOCKER_USER%/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Vérification avec logs') {
            steps {
                bat 'docker ps'
                bat 'docker logs backend-app'
            }
        }
    }
}