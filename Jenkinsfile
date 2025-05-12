pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'e752556d-0bc6-4985-ad16-6f2a663ce000'
        DOCKER_HUB_REPO = 'nadinc/guvi_final_prj'
        IMAGE_NAME = "${DOCKER_HUB_REPO}"
        NODE_ENV = 'production'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/nadin-c/ReactDevops.git', branch: 'main'
            }
        }

        stage('Generate Tag') {
            steps {
                script {
                    env.TAG = "${env.BUILD_NUMBER}-${new Date().format('yyyyMMdd-HHmmss')}"
                    echo "📌 Generated TAG: ${env.TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Tag with "reduced-size"
                    sh "docker tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:reduced-size"

                    // Push both tags
                    sh "docker push ${IMAGE_NAME}:${TAG}"
                    sh "docker push ${IMAGE_NAME}:reduced-size"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded!'
            echo "✅ Docker image pushed with tags:"
            echo "   🔹 ${IMAGE_NAME}:${TAG}"
            echo "   🔹 ${IMAGE_NAME}:reduced-size"
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
