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
                    // Use "${params.GIT_BRANCH}" instead of ${params.BRANCH_NAME}
                    git branch: "${params.GIT_BRANCH}", credentialsId: 'github-cred-id', url: "${params.GIT_URL}"
                }
            }
        }

        stage('Read Properties') {
            steps {
                script {
                    def props = readProperties file: 'app/jenkins.properties'
                    // Assign to environment variables properly
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
                script {
                    sh """
                        echo "${env.DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${env.DOCKERHUB_CREDENTIALS_USR}" --password-stdin
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
