tools {
    // This must match the name you gave it in Global Tool Configuration
    sonarScanner 'SonarScanner' 
}
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
            withSonarQubeEnv('sonar') {
                // Use the standalone scanner instead of Gradle
                bat "sonar-scanner"
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
