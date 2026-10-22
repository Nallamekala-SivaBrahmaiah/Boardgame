pipeline {
    agent any
    tools {
        jdk 'java-17'
        maven 'maven-3.9'
    }

    stages {
        stage('git clone') {
            steps {
                git branch: 'main', url: 'https://github.com/Nallamekala-SivaBrahmaiah/Boardgame.git'
            }
        }
        stage('maven compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('maven package') {
            steps {
                sh 'mvn package'
            }
        }         
    }
}
