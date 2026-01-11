pipeline {
    agent any

    tools {
        jdk 'jdk8' // Use the installed JDK 8
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/chaimabouneb/tp7_ogl.git', branch: 'main'
            }
        }

        stage('Build & Test') {
            steps {
                bat './gradlew clean test'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONARQUBE = 'sonar' // your SonarQube configuration name in Jenkins
            }
            steps {
                withSonarQubeEnv('sonar') {
                    bat './gradlew sonarqube'
                }
            }
        }
    }

    post {
        always {
            junit 'build/test-results/test/**/*.xml'
        }
    }
}
