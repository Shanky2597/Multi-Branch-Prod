pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "deathnote401/multibranch-flask-app"
        IMAGE_TAG  = "build-${BUILD_NUMBER}"
        GIT_USER   = "Shanky2597"
        GIT_EMAIL  = "sanketshinde1404@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
                    """
                }
            }
        }

        stage('Update K8s Manifest') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh """
                    set -e
                    git config user.name "$GIT_USER"
                    git config user.email "$GIT_EMAIL"

                    git fetch origin
                    git checkout main
                    git reset --hard origin/main

                    sed -i.bak "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yaml

                    git add k8s/deployment.yaml
                    git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"

                    set +x
                    git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/Shanky2597/Multi-Branch-Prod.git main
                    """
                }
            }
        }
    }
}
