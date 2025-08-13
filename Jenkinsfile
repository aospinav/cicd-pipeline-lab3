pipeline {
    agent any

    tools {
        nodejs 'node-7.8.0'
    }

    environment {
        // Definir nombre de imagen y puerto por rama a
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        CONTAINER_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3000'}"  // app escucha en 3000
        HOST_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"       // expuesto en host
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${env.IMAGE_NAME}"
                    sh "docker build -t ${env.IMAGE_NAME} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerName = "app-${env.BRANCH_NAME}"

                    // Detener y eliminar contenedor anterior (bajo downtime)
                    sh """
                        docker stop ${containerName} || true
                        docker rm ${containerName} || true
                    """

                    // Levantar nuevo contenedor
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --name ${containerName} -p 3000:3000 ${env.IMAGE_NAME}"
                    } else {
                        sh "docker run -d --name ${containerName} -p 3001:3000 ${env.IMAGE_NAME}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Aplicación desplegada en http://localhost:${env.HOST_PORT}"
        }
        failure {
            echo "❌ Fallo en el pipeline"
        }
    }
}