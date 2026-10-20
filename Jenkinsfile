
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: '1.0.0-1',
            description: 'Docker image tag built and pushed by CI'
        )
    }

    environment {
        DOCKER_USER = 'phuson0206'
        GITOPS_BRANCH = 'main'
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout GitOps Repository') {
            steps {
                checkout scm
                sh '''
                    echo "GitOps repository checked out"
                    git status --short
                    ls -la base/
                '''
            }
        }

        stage('Update Deployment Image Tags') {
            steps {
                sh '''
                    set -eu

                    BACKEND_FILE="base/backend-deployment.yaml"
                    FRONTEND_FILE="base/frontend-deployment.yaml"

                    test -f "$BACKEND_FILE"
                    test -f "$FRONTEND_FILE"

                    sed -i "s|image: ${DOCKER_USER}/car-marketplace-backend:.*|image: ${DOCKER_USER}/car-marketplace-backend:${IMAGE_TAG}|" "$BACKEND_FILE"

                    sed -i "s|image: ${DOCKER_USER}/car-marketplace-frontend:.*|image: ${DOCKER_USER}/car-marketplace-frontend:${IMAGE_TAG}|" "$FRONTEND_FILE"

                    echo "Backend image:"
                    grep 'image:' "$BACKEND_FILE"

                    echo "Frontend image:"
                    grep 'image:' "$FRONTEND_FILE"
                '''
            }
        }

        stage('Commit and Push GitOps') {
            steps {
                withCredentials([
                    gitUsernamePassword(
                        credentialsId: 'gitops-tokens',
                        gitToolName: 'Default'
                    )
                ]) {
                    sh '''
                        set -eu

                        git config user.name "PhuSon26"
                        git config user.email "leson2185@gmail.com"

                        git add \
                            base/backend-deployment.yaml \
                            base/frontend-deployment.yaml

                        if git diff --cached --quiet; then
                            echo "No manifest changes to commit"
                        else
                            git commit -m "deploy: update XeHay images to ${IMAGE_TAG}"
                        fi

                        git push origin HEAD:${GITOPS_BRANCH}
                    '''
                }
            }
        }
    }

    post {
        success {
            script {
                def message = """
    :rocket: *XeHay CD SUCCESS*
    *Job:* ${env.JOB_NAME}
    *Build:* #${env.BUILD_NUMBER}
    *Status:* GitOps manifests updated successfully
    *Build URL:* ${env.BUILD_URL}
    """.stripIndent().trim()
    
                try {
                    withCredentials([
                        string(
                            credentialsId: 'slack-webhook',
                            variable: 'SLACK_WEBHOOK_URL'
                        )
                    ]) {
                        writeFile(
                            file: 'slack-payload.json',
                            text: groovy.json.JsonOutput.toJson([text: message])
                        )
    
                        sh '''
                            set +x
                            curl --fail --silent --show-error \
                              --connect-timeout 10 \
                              --max-time 20 \
                              -X POST \
                              -H 'Content-Type: application/json' \
                              --data-binary @slack-payload.json \
                              "$SLACK_WEBHOOK_URL"
                        '''
                    }
                } catch (err) {
                    echo "Slack notification failed: ${err.getMessage()}"
                }
            }
        }
    
        failure {
            script {
                def message = """
    :rotating_light: *XeHay CD FAILED*
    *Job:* ${env.JOB_NAME}
    *Build:* #${env.BUILD_NUMBER}
    *Build URL:* ${env.BUILD_URL}
    """.stripIndent().trim()
    
                try {
                    withCredentials([
                        string(
                            credentialsId: 'slack-webhook',
                            variable: 'SLACK_WEBHOOK_URL'
                        )
                    ]) {
                        writeFile(
                            file: 'slack-payload.json',
                            text: groovy.json.JsonOutput.toJson([text: message])
                        )
    
                        sh '''
                            set +x
                            curl --fail --silent --show-error \
                              --connect-timeout 10 \
                              --max-time 20 \
                              -X POST \
                              -H 'Content-Type: application/json' \
                              --data-binary @slack-payload.json \
                              "$SLACK_WEBHOOK_URL"
                        '''
                    }
                } catch (err) {
                    echo "Slack notification failed: ${err.getMessage()}"
                }
            }
        }
    
        always {
            echo "CD pipeline finished."
        }
    }
}
