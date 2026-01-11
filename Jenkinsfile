pipeline {
    agent any

    tools {
        jdk 'jdk17'   // 🔥 FORCE Java 17 for ALL stages
    }

    stages {

        stage('Test') {
            steps {
                bat 'java -version'
                bat './gradlew clean test'
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    bat 'java -version'
                    bat './gradlew sonarqube'
                }
            }
        }

        stage('Code Quality') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
