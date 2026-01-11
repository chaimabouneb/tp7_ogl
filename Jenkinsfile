pipeline {
    agent any

    environment {
        // Use the SonarQube token credential stored in Jenkins
        SONARQUBE_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Checkout code from Git
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                // Use Gradle wrapper directly
                bat './gradlew clean test jacocoTestReport'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Run SonarQube analysis using the token
                bat """
                ./gradlew sonarqube ^
                    -Dsonar.projectKey=tp7_ogl ^
                    -Dsonar.host.url=http://YOUR_SONARQUBE_SERVER:9000 ^
                    -Dsonar.login=%SONARQUBE_TOKEN%
                """
            }
        }
    }

    post {
        always {
            // Publish test results if they exist
            junit '**/build/test-results/test/*.xml'
            
            // Publish JaCoCo code coverage
            jacoco execPattern: '**/build/jacoco/test.exec', classPattern: '**/build/classes/java/main', sourcePattern: '**/src/main/java'
            
            echo 'Pipeline finished. Check console output and reports.'
        }
        failure {
            echo 'Build failed! Check console output and test reports.'
        }
    }
}
