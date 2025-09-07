pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "bharath575/hello-world"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-creds', serverUrl: 'https://<K8S-API-ENDPOINT>']) {
                    sh """
                      kubectl set image deployment/hello-world hello-world=$DOCKER_IMAGE:$BUILD_NUMBER --record || kubectl apply -f k8s-deployment.yaml
                    """
                }
            }
        }
    }
}
