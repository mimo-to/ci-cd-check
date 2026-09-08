pipeline {
    agent any

    environment {
        IMAGE_NAME = "mimo017/ci-cd-check"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
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
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                sshagent(['github-ssh']) {
                    sh '''
                        rm -rf manifest-repo

                        git clone git@github.com:mimo-to/ci-cd-check-manifest.git manifest-repo

                        sed -i "s|image: .*|image: mimo017/ci-cd-check:${IMAGE_TAG}|g" \
                        manifest-repo/k8s/deployment.yaml

                        cd manifest-repo

                        git config user.email "jenkins@local"
                        git config user.name "Jenkins"

                        git add .
                        git commit -m "Update image tag to ${IMAGE_TAG}" || true

                        git push origin main
                    '''
                }
            }
        }
    }
}