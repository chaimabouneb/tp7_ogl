pipeline {
    agent any

    environment {
        SONARQUBE_TOKEN = credentials('sonar-token') // use your Jenkins credential ID
    }

    tools {
        // Make sure the names match your Jenkins global tools configuration
        gradle 'Gradle8'
        jdk 'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                bat './gradlew clean test jacocoTestReport'
            }
            post {
                always {
                    junit '**/build/test-results/test/*.xml'
                    jacoco execPattern: '**/build/jacoco/test.exec'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Make sure the SonarQube server name matches your Jenkins config
                    bat './gradlew sonarqube'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed! Check console output and test reports.'
        }
    }
}
