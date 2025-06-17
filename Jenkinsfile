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
                    git branch: $env.BRANCH_NAME, credentialsId: 'github-cred-id', url: $env.GIT_URL
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $env.IMAGE_NAME:$env.IMAGE_TAG ."
                }
            }
        }

        stage('Login to Docker Hub & Push Image') {
            steps {
                script {
                    sh """
                        echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                        docker push ${env.IMAGE_NAME}:${env.IMAGE_TAG}
                    """
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
