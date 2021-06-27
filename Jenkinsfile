pipeline {
    agent {
        label 'docker'
    }

    environment {
        IMAGE_REPO = 'wordprove/java'
        TAG = sh(script: "jq -r /version package.json", returnStdout: true).trim()
    }

    stages {
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
        stage('Build') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh '''
                    printenv
                    docker build -t $IMAGE_REPO:$TAG-$GIT_COMMIT -t $IMAGE_REPO:latest ./
                    docker push $IMAGE_REPO:$TAG-$GIT_COMMIT
                    docker push $IMAGE_REPO:latest
                    '''
                }
            }
        }
        stage('Deploy') {
            environment {
                NAMESPACE = 'pyrko'
            }
            steps {
                sh '''
                tmpfile=$(mktemp)
                for i in kubernetes/*.yaml; do
                    cat $i | envsubst > $tmpfile
                    cp -pf $tmpfile $i
                    rm -f "$tmpfile"
                done
                '''

                withCredentials([kubeconfigFile(credentialsId: 'c3b37b51-5f32-4fac-8e1e-031468632cee', variable: 'KUBECONFIG')]) {
                    sh 'kubectl -n $NAMESPACE apply -f kubernetes/'
                }
            }
        }
    }
}