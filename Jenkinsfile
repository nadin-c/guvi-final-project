pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "nadinc/guvi_final_prj"
        TAG = "${BUILD_NUMBER}-${sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()}"
        CONTAINER_NAME = "jenkins-docker-container"
        PORT = "8080"
        DOCKER_CREDENTIALS_ID = 'docker-hub-creds'
        NODE_ENV = 'production'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/nadin-c/guvi-final-project.git', branch: 'main'
            }
        }

        stage('Build Application') {
            steps {
                script {
                    sh 'npm install --production'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build \
                      --build-arg BUILD_DATE=\$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
                      --build-arg VERSION=${TAG} \
                      -t ${DOCKER_HUB_REPO}:latest \
                      -t ${DOCKER_HUB_REPO}:${TAG} .
                    """
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh """
                    docker push ${DOCKER_HUB_REPO}:latest
                    docker push ${DOCKER_HUB_REPO}:${TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
            echo 'Docker image pushed to Docker Hub!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            sh 'docker logout'
        }
    }
}
