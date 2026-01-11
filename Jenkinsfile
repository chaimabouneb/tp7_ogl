pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                bat './gradlew clean compileJava compileTestJava'
            }
        }
        
        stage('Test') {
            steps {
                bat './gradlew test'
            }
            post {
                always {
                    junit 'build/test-results/test/**/*.xml'
                    cucumber jsonReportDirectory: 'build/reports/cucumber/', fileIncludePattern: '*.json'
                }
            }
        }
     
        stage('Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        bat './gradlew sonarqube'
                    }
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }
    
    // Add this to ensure workspace is cleaned
    post {
        always {
            cleanWs()
        }
    }
}
