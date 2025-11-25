pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "majd04/app"            // ← mets le nom de ton image Docker
        DOCKER_TAG   = "latest"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Majd04bougatef/Majd-Bougatef-4SIM1.git'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    """
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh """
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès ! 🚀'
        }
        failure {
            echo 'Une erreur est survenue ❌'
        }
    }
}
