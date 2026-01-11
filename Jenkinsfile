pipeline {
    agent any

    // 1️⃣ Set environment variables
    environment {
        SONARQUBE_TOKEN = credentials('sonar-token') // Secret Text from Jenkins credentials
    }

    tools {
        jdk 'jdk8' // Make sure you configured JDK 8 in Jenkins
    }

    stages {

        stage('Checkout') {
            steps {
                // Checkout the Git repository
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                // Use Gradle wrapper to build and run tests
                script {
                    if (isUnix()) {
                        sh './gradlew clean test'
                    } else {
                        bat './gradlew clean test'
                    }
                }
            }

            // Always archive test results even if build fails
            post {
                always {
                    junit 'build/test-results/test/**/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis using the Gradle wrapper and token
                script {
                    if (isUnix()) {
                        sh "./gradlew sonarqube -Dsonar.login=${env.SONARQUBE_TOKEN}"
                    } else {
                        bat "./gradlew sonarqube -Dsonar.login=${env.SONARQUBE_TOKEN}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build and analysis completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
