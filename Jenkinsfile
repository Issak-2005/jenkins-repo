pipeline {
    agent any

    parameters {
        string(name: 'GIT_URL', description: 'GitHub repository URL')
        string(name: 'BRANCH_NAME',  description: 'Git branch to clone')
        string(name: 'IMAGE_NAME',  description: 'Docker image name (e.g., user/image)')
        string(name: 'IMAGE_TAG', defaultValue: "latest", description: 'Docker image tag (e.g., latest)')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred-id')
    }

    stages {
        stage('Read Properties') {
            steps {
                script {
                     // Load properties from the file in the workspace
                    def props = readProperties file: 'jenkins.properties'

                    // // Access individual properties
                    // echo "ENV = ${props['ENV']}"
                    // echo "REGION = ${props['REGION']}"
                    // echo "TIMEOUT = ${props['TIMEOUT']}"

                    // You can also assign them to environment variables if needed
                    env.GIT_URL = props['GIT_URL']
                    env.BRANCH_NAME=props['BRANCH_NAME']
                    env.IMAGE_NAME=props['IMAGE_NAME']
                }
            }
        }
        stage('Clone Repo') {
            steps {
                git branch: "${params.BRANCH_NAME}", credentialsId: 'github-cred-id', url: "${params.GIT_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${params.IMAGE_NAME}:${params.IMAGE_TAG} ."
                }
            }
        }

        stage('Login to Docker Hub & Push Image') {
            steps {
                script {
                    sh """
                        echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                        docker push ${params.IMAGE_NAME}:${params.IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed: ${params.IMAGE_NAME}:${params.IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
