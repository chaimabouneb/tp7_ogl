pipeline {
    agent any

    tools {
        // Use JDK 8 installed on Jenkins
        jdk 'jdk8'
        // SonarQube scanner (must be configured in Jenkins global tools)
        sonarScanner 'SonarQubeScanner'
    }

    environment {
        // SonarQube server authentication token
        SONARQUBE_TOKEN = credentials('sonarqube-token') 
        // Update this URL to match your SonarQube server
        SONAR_HOST_URL = 'http://your-sonarqube-server:9000'
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
                    jacoco execPattern: 'build/jacoco/test.exec'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') { // Name of the SonarQube server configured in Jenkins
                    bat "./gradlew sonarqube " +
                        "-Dsonar.host.url=%SONAR_HOST_URL% " +
                        "-Dsonar.login=%SONARQUBE_TOKEN%"
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
            archiveArtifacts artifacts: 'build/reports/jacoco/test/jacocoTestReport.xml', allowEmptyArchive: true
            echo 'Pipeline finished.'
        }
    }
}
