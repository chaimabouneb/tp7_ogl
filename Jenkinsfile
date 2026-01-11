pipeline {
    agent any

    tools {
        jdk 'jdk8' // Ensure JDK 8 installed on Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/chaimabouneb/tp7_ogl.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat './gradlew clean test'
            }
            post {
                always {
                    junit 'build/test-results/test/**/*.xml'
                }
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') { // Must match Jenkins SonarQube server name
                    bat './gradlew sonarqube'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/reports/jacoco/test/html/**', allowEmptyArchive: true
        }
    }
}
