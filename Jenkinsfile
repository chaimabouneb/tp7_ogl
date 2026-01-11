pipeline {
    agent any

    environment {
        SONARQUBE_TOKEN = credentials('sonar-token')
    }

    tools {
        gradle 'Gradle8'   // Keep your working Gradle
        jdk 'jdk17'        // Match your Jenkins JDK installation name
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
                    // Comment this out or delete it if the plugin isn't installed
                    // jacoco execPattern: '**/build/jacoco/test.exec' 
                }
            }
        }

   stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        // Use 'sonar' instead of 'sonarqube' (it's the newer command)
                        bat './gradlew sonar' 
                    }
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
