pipeline {
    agent any

    tools {
        // Use the JDK 8 installation configured in Jenkins
        jdk 'jdk8'
        // Gradle installed in Jenkins (optional, Gradle wrapper will be used)
        gradle 'Gradle_8.14'
    }

    environment {
        SONARQUBE_TOKEN = credentials('sonarqube-token') // Jenkins credentials ID
        SONAR_HOST_URL = 'http://your-sonarqube-server:9000' // Update to your server
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/chaimabouneb/tp7_ogl.git'
            }
        }

        stage('Build & Test') {
            steps {
                // Use Gradle wrapper
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
                // Use Gradle SonarQube plugin
                bat "./gradlew sonarqube " +
                    "-Dsonar.host.url=%SONAR_HOST_URL% " +
                    "-Dsonar.login=%SONARQUBE_TOKEN%"
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
