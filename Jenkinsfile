pipeline {
    agent any

    tools {
        jdk 'jdk8'   // Ensure JDK 8 is installed on your Jenkins node
    }

    environment {
        SONARQUBE_TOKEN = credentials('sonar-token') // Replace with your Jenkins SonarQube token ID
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/chaimabouneb/tp7_ogl.git', branch: 'main'
            }
        }

        stage('Build & Test') {
            steps {
                // Use Gradle wrapper instead of Jenkins Gradle tool
                bat './gradlew clean test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis with Gradle wrapper
                bat './gradlew sonarqube'
            }
        }
    }

    post {
        always {
            // Archive test results
            junit 'build/test-results/test/**/*.xml'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
