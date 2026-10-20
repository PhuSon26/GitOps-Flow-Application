pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: '1.0.0-1',
            description: 'Docker image tag to deploy'
        )
    }

    environment {
        DOCKER_USER = 'phuson0206'
        GITOPS_BRANCH = 'main'
    }

    stages {
        stage('Update Image Tags') {
            steps {
                sh '''
                    set -eu

                    sed -i "s|image: ${DOCKER_USER}/car-marketplace-backend:.*|image: ${DOCKER_USER}/car-marketplace-backend:${IMAGE_TAG}|" base/backend-deployment.yaml

                    sed -i "s|image: ${DOCKER_USER}/car-marketplace-frontend:.*|image: ${DOCKER_USER}/car-marketplace-frontend:${IMAGE_TAG}|" base/frontend-deployment.yaml

                    echo "Updated backend:"
                    grep 'image:' base/backend-deployment.yaml

                    echo "Updated frontend:"
                    grep 'image:' base/frontend-deployment.yaml
                '''
            }
        }

        stage('Commit and Push GitOps') {
            steps {
                withCredentials([gitUsernamePassword(
                    credentialsId: 'Github-tokens',
                    gitToolName: 'Default'
                )]) {
                    sh '''
                        set -eu
                        git config user.name "PhuSon26"

                        git config user.email "leson2185@gmail.com"

                        git add base/backend-deployment.yaml base/frontend-deployment.yaml

                        git commit -m "deploy: update XeHay images to ${IMAGE_TAG}"
                        git push origin HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'GitOps update pushed. Argo CD will synchronize the new image tags.'
        }

        failure {
            echo 'CD failed. Check Console Output.'
        }
    }
}
