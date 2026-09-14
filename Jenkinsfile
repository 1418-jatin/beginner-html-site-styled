pipeline {
    agent { label 'linux agent' }   // exactly same as agent config

    environment {
        IMAGE_NAME = "19901418/my-jenkins-python-app-ci-cd"
        IMAGE_TAG  = "v1"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/1418-jatin/beginner-html-site-styled.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl delete deployment beginner-html-deployment --ignore-not-found=true
                kubectl delete service beginner-html-service --ignore-not-found=true

                kubectl apply -f deployment.yml
                kubectl apply -f service.yml
                '''
            }
        }
    }

    triggers {
        githubPush()
    }
}
