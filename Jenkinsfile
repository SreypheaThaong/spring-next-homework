pipeline {
    agent any
    environment {
        REGISTRY = "solenn9/spring-boot"
        IMAGE_TAG = "${BUILD_NUMBER}"
        HELM_REPO = "https://github.com/Solen-s/Manifest-Spring-boot.git"
        HELM_VALUES_FILE = "values.yaml"
    }
    stages {
        stage('Checkout App') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'fc770254-9dd1-4ad5-981f-1c0d225bf802', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    script {
                        try {
                            sh """
                                echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                                docker build -t ${REGISTRY}:${IMAGE_TAG} .
                                docker push ${REGISTRY}:${IMAGE_TAG}
                                docker logout
                            """
                            echo "✅ Docker image built and pushed successfully: ${REGISTRY}:${IMAGE_TAG}"
                        } catch (err) {
                            echo "❌ Docker build/push failed!"
                            error("Stopping pipeline due to Docker error.")
                        }
                    }
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                withCredentials([usernamePassword(credentialsId: '367a1978-14ee-4cbc-acf4-8c9467428168', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    script {
                        try {
                            sh """
                                rm -rf helm-spring-boot-repo || true
                                git clone ${HELM_REPO} helm-spring-boot-repo
                                cd helm-spring-boot-repo
                                sed -i 's|tag:.*|tag: "${IMAGE_TAG}"|' ${HELM_VALUES_FILE}
                                git add ${HELM_VALUES_FILE}
                                git commit -m "Update image tag to ${IMAGE_TAG}"
                                git push origin main
                            """
                            echo "✅ Helm values updated and pushed successfully."
                        } catch (err) {
                            echo "❌ Updating Helm values failed!"
                            error("Stopping pipeline due to Git/Helm error.")
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "💥 Pipeline failed. Check logs above."
        }
    }
}