pipeline {
    agent any

    environment {
        // use Jenkins credentials IDs
        DOCKER_CREDENTIALS = credentials('dockerhub-token')
        GITHUB_CREDENTIALS = credentials('github-token')
        IMAGE = "phea12/spring-homework-image"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $IMAGE:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    try {
                        // Docker login using Jenkins credential env vars
                        if (sh(script: 'echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin', returnStatus: true) == 0) {
                            echo "✅ Docker login successful."

                            if (sh(script: "docker push $IMAGE:${BUILD_NUMBER}", returnStatus: true) == 0) {
                                echo "✅ Docker image pushed successfully."
                            } else {
                                echo "❌ Docker push failed!"
                                error("Docker push failed.")
                            }

                        } else {
                            echo "❌ Docker login failed!"
                            error("Login to Docker registry failed.")
                        }
                    } catch (err) {
                        echo "❌ Error during Docker push stage: ${err}"
                        error("Stage failed due to above error.")
                    }
                }
            }
        }

    }
}