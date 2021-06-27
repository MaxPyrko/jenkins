pipeline {
    agent {
        label 'docker'
    }

    environment {
        IMAGE_REPO = 'wordprove/java'
        TAG = '1.0.0'
    }

    stages {
        stage('Build') {
            steps {
                sh 'printenv'
                sh 'docker build -t $IMAGE_REPO:$TAG-$GIT_COMMIT -t $IMAGE_REPO:latest ./'
            }
        }
    }
}