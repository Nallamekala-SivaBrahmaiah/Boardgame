
pipeline {
    agent any

    tools {
        jdk 'java-17'
        maven 'maven-3.9'
    }

    environment {
        DOCKER_IMAGE = 'nsbyadav14e/boardgame'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Nallamekala-SivaBrahmaiah/Boardgame.git'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Maven Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
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

        stage('Docker Push') {
            steps {
                sh '''
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    docker push $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker rm -f boardgame-app 2>/dev/null || true

                    docker run -d \
                        --name boardgame-app \
                        --restart unless-stopped \
                        -p 8081:8080 \
                        $DOCKER_IMAGE:$BUILD_NUMBER
                '''
            }
        }
    }
}
