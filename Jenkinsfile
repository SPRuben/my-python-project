pipeline {
    agent any

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
