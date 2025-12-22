
pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
      
        DOCKERHUB_USER = 'majd04bougatef'
        IMAGE_NAME = 'app'
        K8S_NAMESPACE = 'devops'
        K8S_MANIFEST_DIR = 'k8s'
        K8S_DEPLOYMENT_NAME = 'student-management-app'
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Majd04bougatef/Majd-Bougatef-4SIM1.git'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('xsonar') { // uses the global SonarQube server config
        //             sh 'mvn sonar:sonar'
        //         }
        //     }
        // }

        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }



        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }
        
        stage('Kubernetes Deploy spring et mysql ') {
            steps {
                 sh """
                        kubectl apply -f ${K8S_MANIFEST_DIR}/
                        kubectl -n ${K8S_NAMESPACE} rollout restart deployment/${K8S_DEPLOYMENT_NAME}
                    """
            }
        }

         stage('Verify') {
            steps {
                    sh """
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/${K8S_DEPLOYMENT_NAME} --timeout=180s
                        kubectl -n ${K8S_NAMESPACE} get pods -o wide
                        kubectl -n ${K8S_NAMESPACE} get svc
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
        always {
            cleanWs()
        }
    }
}
