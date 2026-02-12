pipeline {
    agent any

    environment {
        // Definimos variables que cambian según la rama
        IMAGE_NAME = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        IMAGE_PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Usamos el script build.sh que me pasaste
                sh 'chmod +x scripts/build.sh'
                sh './scripts/build.sh'
            }
        }

        stage('Test') {
            steps {
                // Usamos el script test.sh que me pasaste
                // Nota: CI=true evita que el test de React se quede bloqueado esperando input
                sh 'chmod +x scripts/test.sh'
                sh 'CI=true ./scripts/test.sh'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Construye la imagen con el nombre condicional (nodemain o nodedev)
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // 1. Detener y eliminar contenedores previos para evitar conflictos de puerto
                    // El lab pide intentar el menor "downtime" posible
                    try {
                        sh "docker stop web-app-${env.BRANCH_NAME} || true"
                        sh "docker rm web-app-${env.BRANCH_NAME} || true"
                    } catch (Exception e) {
                        echo "No había contenedores previos corriendo."
                    }

                    // 2. Correr el nuevo contenedor en el puerto correspondiente (3000 o 3001)
                    // Se usa -p puerto_host:3000 porque el Dockerfile no define EXPOSE, 
                    // pero la app corre en el 3000 por defecto.
                    sh "docker run -d --name web-app-${env.BRANCH_NAME} -p ${IMAGE_PORT}:3000 ${IMAGE_NAME}"
                }
            }
        }
    }
}
