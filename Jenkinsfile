pipeline {
    agent any
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Build') {
            steps {
                bat './gradlew clean compileJava compileTestJava'
            }
        }
        
        stage('Test') {
            steps {
                bat './gradlew test --info'
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
                        bat './gradlew sonarqube -Dsonar.gradle.skipCompile=true'
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
}
