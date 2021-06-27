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
                sh '''
                printenv
                docker build -t $IMAGE_REPO:$TAG-$GIT_COMMIT -t $IMAGE_REPO:latest ./
                '''
            }
        }

        stage('Tests') {
            agent {
                docker {
                    image 'node:14-alpine'
                    label 'docker'
                }
                
            }
            steps {
                sh '''
                npm ci
                npm test
                '''
            }

            post {
                always {
                    recordIssues(
                        enabledForFailure: true, 
                        tools: [
                            junitParser(pattern: 'test-results.xml')
                        ]
                    )
                }
            }
        }
    }
}