
pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        DOCKER_IMAGE = 'nsbyadav14e/boardgame'
        ACR_LOGIN_SERVER = 'nsbyadav1434567.azurecr.io'
        ACR_IMAGE = 'boardgame'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Nallamekala-SivaBrahmaiah/Boardgame.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASS" | docker login \
                            --username "$DOCKER_USER" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t $DOCKER_IMAGE:$BUILD_NUMBER \
                        -t $DOCKER_IMAGE:latest .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    docker push $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Azure ACR Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'azure-acr-creds',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh '''
                        set +x
                        echo "$ACR_PASS" | docker login \
                            "$ACR_LOGIN_SERVER" \
                            --username "$ACR_USER" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push to Azure ACR') {
            steps {
                sh '''
                    docker tag $DOCKER_IMAGE:$BUILD_NUMBER \
                        $ACR_LOGIN_SERVER/$ACR_IMAGE:$BUILD_NUMBER

                    docker tag $DOCKER_IMAGE:latest \
                        $ACR_LOGIN_SERVER/$ACR_IMAGE:latest

                    docker push $ACR_LOGIN_SERVER/$ACR_IMAGE:$BUILD_NUMBER
                    docker push $ACR_LOGIN_SERVER/$ACR_IMAGE:latest
                '''
            }
        }
    }
}
