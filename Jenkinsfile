pipeline {
    agent { label 'linux agent' }   // run only on your EC2 Jenkins agent

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

        stage('Run Container on Port 99') {
            steps {
                sh '''
                    # Stop and remove old container if it exists
                    docker rm -f myapp || true

                    # Run new container mapping host port 99 -> container port 80
                    docker run -d --name myapp -p 99:80 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    triggers {
        githubPush()
    }
}
