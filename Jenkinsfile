pipeline {
    agent any

    stages {

        stage('Test') {
            steps {
                bat './gradlew'
            }
            post {
                always {
                    junit 'build/test-results/test/**/*.xml'
                }
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
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
