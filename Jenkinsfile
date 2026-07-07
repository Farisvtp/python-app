pipeline {
    agent any

    environment {
        IMAGE_NAME = "faris7/python-app"
        IMAGE_TAG = "${BUILD_NUMBER}"

        APP_REPO = "https://github.com/Farisvtp/python-app.git"
        GITOPS_REPO = "https://github.com/Farisvtp/python-app-gitops.git"
    }

    stages {

        stage('Checkout Application') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: "${APP_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Update GitOps Repository') {
            steps {
                dir('python-app-gitops') {

                    git branch: 'main',
                        credentialsId: 'github-credentials',
                        url: "${GITOPS_REPO}"

                    withCredentials([usernamePassword(
                        credentialsId: 'github-credentials',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {

                        sh """
                            sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml

                            git config user.name "Jenkins"
                            git config user.email "jenkins@local"

                            git add deployment.yaml

                            git diff --cached --quiet || git commit -m "Update image to ${IMAGE_TAG}"

                            git remote set-url origin https://\$GIT_USER:\$GIT_TOKEN@github.com/Farisvtp/python-app-gitops.git

                            git push origin main
                        """
                    }
                }
            }
        }

    }

    post {

        success {
            echo "========================================="
            echo "Docker image built successfully"
            echo "Docker image pushed to Docker Hub"
            echo "GitOps repository updated successfully"
            echo "Argo CD will automatically sync"
            echo "========================================="
        }

        failure {
            echo "Pipeline Failed!"
        }
    }
}
