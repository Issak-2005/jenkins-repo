pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', description: 'Enter Branch Name')
        string(name: 'GIT_URL', description: 'Enter Repo URL')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred-id')
    }

    stages {
        stage('Clone Repo') {
            steps {
                script {
                    git branch: "${params.GIT_BRANCH}",
                        credentialsId: 'github-cred-id',
                        url: "${params.GIT_URL}"
                }
            }
        }

        stage('Read Properties') {
            steps {
                script {
                    def props = readProperties file: 'jenkins.properties'
                    env.IMAGE_NAME = props['IMAGE_NAME']
                    env.IMAGE_TAG  = props['IMAGE_TAG']
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} ."
                }
            }
        }

        stage('Login to Docker Hub & Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cred-id',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    script {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push "$IMAGE_NAME:$IMAGE_TAG"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed: ${env.IMAGE_NAME}:${env.IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
