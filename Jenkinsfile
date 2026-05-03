pipeline {
    agent {
        label 'dockerhost-build-server'
    }

    environment {
        DOCKER_USER = 'jlcendguz'
        IMAGE_NAME = 'mytomcat'
        DOCKER_HUB_CREDS = credentials('devops-dockerhub-token')
    }

    stages {
        stage('Limpieza del Entorno') {
            steps {
                echo 'Limpiando contenedores antiguos...'
                sh "docker rm -f ${IMAGE_NAME} || true"
                sh "docker image prune -f || true"
            }
        }

        stage('Construcción de Imagen') {
            steps {
                echo 'Construyendo la imagen de Tomcat...'
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                sh "docker tag ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Login y Push a Docker Hub') {
            steps {
                echo 'Subiendo imagen a Docker Hub...'
                sh "echo \$DOCKER_HUB_CREDS_PSW | docker login -u \$DOCKER_HUB_CREDS_USR --password-stdin"
                sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Despliegue Local') {
            steps {
                echo 'Ejecutando el contenedor en el Docker Host...'
                sh "docker run -d --name ${IMAGE_NAME} -p 8081:8080 ${DOCKER_USER}/${IMAGE_NAME}:latest"
            }
        }
    }

    post {
        always {
            echo 'Cerrando sesión de Docker...'
            sh "docker logout"
        }
    }
}
