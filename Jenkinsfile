pipeline {
    agent any

    
    parameters {
        // Define a boolean parameter for the user to choose if Docker should run
        booleanParam(name: 'RUN_DOCKER', defaultValue: true, description: 'Run the Docker build and push stage')
    }

    environment {
        DOCKER_LOGIN = 'rubenssp'
    }

    stages {

        stage('Parallel') {
            parallel {
                stage('Flake8') {
                    steps {
                        script {
                            sh 'python3 --version'
                            sh 'python3 -m flake8 --version'
                            sh 'python3 -m flake8 . --count --show-source --statistics || true'
                        }
                    }
                }

                stage('Unit Tests') {
                    steps {
                        script {
                            sh 'pytest | tee report.txt'
                            archiveArtifacts artifacts: 'report.txt', fingerprint: true
                        }
                    }
                }
            
                stage('Docker') {
                    when {
                        expression { return params.RUN_DOCKER } // Only run Docker stage if the user selects 'true'
                    }
                    steps {
                        script {
                            withCredentials([string(credentialsId: 'DOCKER_PASSWORD_RS', variable: 'DOCKER_PASS')]) {
                                sh 'docker build -t rubenssp/my-python:$BUILD_NUMBER .'
                                sh 'echo $DOCKER_PASS | docker login -u $DOCKER_LOGIN --password-stdin'
                                sh 'docker push rubenssp/my-python:$BUILD_NUMBER'
                            }
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
