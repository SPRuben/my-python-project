pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_DOCKER', defaultValue: true, description: 'Run the Docker build and push stage')
    }

    environment {
        DOCKER_LOGIN = 'rubenssp'
        DOCKER_IMAGE = 'rubenssp/my-python'  // Store the Docker image name in a variable
        DOCKER_IMAGE_TAG = "$BUILD_NUMBER"   // Tag for Docker image based on the build number
    }

    stages {
        stage('Parallel') {
            parallel {
                stage('Flake8') {
                    steps {
                        sh '''
                            python3 --version
                            python3 -m flake8 --version
                            python3 -m flake8 . --count --show-source --statistics || true
                        '''
                    }
                }

                stage('Unit Tests') {
                    steps {
                        sh '''
                            pytest | tee report.txt
                        '''
                        archiveArtifacts artifacts: 'report.txt', fingerprint: true
                    }
                }

                stage('Docker') {
                    when {
                        expression { return params.RUN_DOCKER }  // Only run if the user selects 'true'
                    }
                    steps {
                        withCredentials([string(credentialsId: 'DOCKER_PASSWORD_RS', variable: 'DOCKER_PASS')]) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_LOGIN --password-stdin
                                docker build -t $DOCKER_IMAGE:$DOCKER_IMAGE_TAG .
                                docker push $DOCKER_IMAGE:$DOCKER_IMAGE_TAG
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline: FAILURE!"
        }
        success {
            echo "Pipeline: DONE!"
        }
    }
}
