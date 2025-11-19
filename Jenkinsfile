pipeline {
    agent any

    stages {
        stage('Git') {
            steps {
                git branch: 'master', url: 'https://github.com/Majd04bougatef/Majd-Bougatef-4SIM1.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès !'
        }
        failure {
            echo 'Erreur dans le pipeline.'
        }
    }
}



