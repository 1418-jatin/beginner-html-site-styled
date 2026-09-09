stage('Build Docker Image') {
    steps {
        sh 'docker build -t beginner-html-site:v1 .'
    }
}

stage('Run Container') {
    steps {
        sh '''
        docker rm -f beginner-html || true
        docker run -dit --name beginner-html -p 99:80 beginner-html-site:v1
        '''
    }
}
