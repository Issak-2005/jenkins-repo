pipeline {
    agent any

  parameters {
        string(name: 'GIT_BRANCH', description: 'Enter Branch Name')
        string(name: 'GIT_URL', description: 'Enter Repo Url')
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred-id')
    }

    stages {
        stage('Clone Repo') {
            steps {
                script {
                    // Using env vars in script block
                    git branch: ${params.BRANCH_NAME}, credentialsId: 'github-cred-id', url: ${params.GIT_URL}
                }
            }
        }
        stage('Read Properties') {
            steps {
                script {
                    // Load properties from file
                    def props = readProperties file: 'app/jenkins.properties'

                    // Set to environment for use in other stages
                    env.IMAGE_NAME  = props['IMAGE_NAME']
                    env.IMAGE_TAG   = props['IMAGE_TAG']
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
                    sh """
                        echo "${env.DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${env.DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """  
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
