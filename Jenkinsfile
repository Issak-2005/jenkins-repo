pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred-id')
    }

    stages {
        stage('Read Properties') {
            steps {
                script {
                    // Load properties from file
                    def props = readProperties file: 'jenkins.properties'

                    // Set to environment for use in other stages
                    env.GIT_URL     = props['GIT_URL']
                    env.BRANCH_NAME = props['BRANCH_NAME']
                    env.IMAGE_NAME  = props['IMAGE_NAME']
                    env.IMAGE_TAG   = props['IMAGE_TAG']
                }
            }
        }

        stage('Clone Repo') {
            steps {
                script {
                    // Using env vars in script block
                    git branch: $BRANCH_NAME, credentialsId: 'github-cred-id', url: $GIT_URL
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
                }
            }
        }

        stage('Login to Docker Hub & Push Image') {
            steps {
                 script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }

            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
