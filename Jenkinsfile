pipeline {
    agent any

    environment {
        IMAGE_NAME = "faris7/python-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        GITOPS_REPO = "https://github.com/Farisvtp/python-app-gitops.git"
    }

    stages {

        stage('Checkout Application') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Farisvtp/python-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                    '''
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
                usernameVariable: 'GIT_USERNAME',
                passwordVariable: 'GIT_TOKEN'
            )]) {

                sh """
                    sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml

                    git config user.name "Jenkins"
                    git config user.email "jenkins@local"

                    git add deployment.yaml
                    git commit -m "Update image to ${IMAGE_TAG}" || true

                    git remote set-url origin https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/Farisvtp/python-app-gitops.git

                    git push origin main
                """
                }
            }
        }
    }

    post {
        success {
            echo "Docker image pushed successfully."
            echo "GitOps repository updated."
            echo "Argo CD will automatically sync the new deployment."
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
