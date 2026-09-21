pipeline {
    agent { label 'linux agent' } // must match the label configured on the Jenkins agent node

    environment {
        IMAGE_NAME  = "19901418/my-jenkins-python-app-ci-cd"
        IMAGE_TAG   = "${BUILD_NUMBER}"          // unique tag per build (was fixed "v1")
        K8S_NODE_IP = "3.145.91.11"             // PRIVATE IP of k8s-node (same VPC as ci-agent; doesn't change on stop/start)
    }

    stages {
        // No separate checkout stage needed: "Declarative: Checkout SCM" already
        // checks out this repo because the Jenkinsfile is loaded from Git.

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // kubectl runs on k8s-node via SSH
                withCredentials([sshUserPrivateKey(credentialsId: 'jenkin-cred', keyFileVariable: 'PEM_KEY')]) {
                    sh '''
                        SSH_OPTS="-i $PEM_KEY -o StrictHostKeyChecking=no -o ConnectTimeout=15"

                        # Point the manifest at the image built in this run
                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" deployment.yml

                        ssh $SSH_OPTS ubuntu@${K8S_NODE_IP} "mkdir -p ~/k8s"
                        scp $SSH_OPTS deployment.yml service.yml ubuntu@${K8S_NODE_IP}:~/k8s/

                        # apply = rolling update; no delete, so no downtime and the LoadBalancer is kept
                        ssh $SSH_OPTS ubuntu@${K8S_NODE_IP} "
                            kubectl apply -f ~/k8s/deployment.yml &&
                            kubectl apply -f ~/k8s/service.yml &&
                            kubectl rollout status deployment/beginner-html-deployment --timeout=180s &&
                            kubectl get svc beginner-html-service
                        "
                    '''
                }
            }
        }
    }

    triggers {
        githubPush()
    }
}
