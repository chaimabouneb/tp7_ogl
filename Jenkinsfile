pipeline {
    agent any
    tools { jdk 'jdk8' }
    environment {
        SONARQUBE_TOKEN = credentials('sonar-token') // Ensure this exists in Jenkins
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
            steps {
                bat './gradlew sonarqube'
            }
        }
    }

    post {
        always {
            // MUST be inside a node context
            node {
                junit 'build/test-results/test/**/*.xml'
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
