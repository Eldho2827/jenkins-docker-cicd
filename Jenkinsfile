pipeline {
    agent any

    environment {
        IMAGE_NAME = "eldho10/cicd-node-app"
    }

    stages {
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

       stage('Test') {
    steps {
        sh 'docker images eldho10/cicd-node-app:latest'
        echo 'Image built successfully!'
    }
}

        stage('Deploy') {
            steps {
                sh 'docker stop node-app || true'
                sh 'docker rm node-app || true'
                sh 'docker run -d --name node-app -p 3000:3000 $IMAGE_NAME:latest'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME:latest'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
