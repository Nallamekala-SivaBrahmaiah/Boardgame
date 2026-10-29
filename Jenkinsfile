
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
---

pipeline {
    agent any

    tools {
        jdk 'java-17'
        maven 'maven-3.9'
    }

    environment {
        ACR_LOGIN_SERVER = 'nsbyadav1434567.azurecr.io'
        IMAGE_NAME = 'boardgame'
        CONTAINER_NAME = 'boardgame-app'
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

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t $ACR_LOGIN_SERVER/$IMAGE_NAME:$BUILD_NUMBER \
                        -t $ACR_LOGIN_SERVER/$IMAGE_NAME:latest .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:$BUILD_NUMBER
                    docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:latest
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    docker rm -f $CONTAINER_NAME 2>/dev/null || true

                    docker run -d \
                        --name $CONTAINER_NAME \
                        --restart unless-stopped \
                        -p 8081:8080 \
                        $ACR_LOGIN_SERVER/$IMAGE_NAME:$BUILD_NUMBER
                '''
            }
        }
    }
}
