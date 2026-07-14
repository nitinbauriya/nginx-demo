pipeline {

    agent any

    environment {

        IMAGE_NAME = "bauriyanitin/nginx-demo"

        IMAGE_TAG = "${BUILD_NUMBER}"

    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD')]) {

                    sh '''
                    echo $PASSWORD | docker login -u $USERNAME --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {

                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'

                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest'

                sh 'docker push $IMAGE_NAME:latest'
            }
        }

        stage('Update Deployment') {
            steps {

                sh '''
                sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" k8s/deployment.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                sh '''
                kubectl apply -f k8s/namespace.yaml

                kubectl apply -f k8s/deployment.yaml

                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {

                sh 'kubectl rollout status deployment/nginx-demo -n nginx-demo'

                sh 'kubectl get pods -n nginx-demo'

                sh 'kubectl get svc -n nginx-demo'
            }
        }

    }

}
