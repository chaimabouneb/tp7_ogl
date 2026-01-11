pipeline {
    agent any

    environment {
        // Path to Gradle wrapper
        GRADLE_CMD = './gradlew'
        // SonarQube token
        SONARQUBE_TOKEN = credentials('SONARQUBE_TOKEN')
    }

    options {
        // Keep only 10 builds to save space
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Timestamps in logs
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    url: 'https://github.com/chaimabouneb/tp7_ogl.git',
                    branch: 'main'
                )
            }
        }

        stage('Build & Test') {
            steps {
                script {
                    // Use offline mode if dependencies are pre-cached
                    // The --offline flag ensures Gradle doesn't try to fetch missing dependencies
                    bat "${env.GRADLE_CMD} clean test --offline"
                }
            }
            post {
                always {
                    // Archive test reports if they exist
                    junit allowEmptyResults: true, testResults: 'build/test-results/test/*.xml'
                }
            }
        }

        stage('Code Coverage') {
            steps {
                script {
                    bat "${env.GRADLE_CMD} jacocoTestReport --offline"
                }
                publishHTML(target: [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'build/reports/jacoco/test/html',
                    reportFiles: 'index.html',
                    reportName: 'JaCoCo Test Report'
                ])
            }
        }

        stage('SonarQube Analysis') {
            when {
                // Only run SonarQube if build and tests succeed
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            environment {
                SONAR_HOST_URL = 'http://YOUR_SONARQUBE_SERVER:9000'
            }
            steps {
                script {
                    bat "${env.GRADLE_CMD} sonarqube " +
                        "-Dsonar.login=${env.SONARQUBE_TOKEN} " +
                        "--offline"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Status: ${currentBuild.currentResult}"
        }
        failure {
            echo "Build failed! Check console output and test reports."
        }
    }
}
