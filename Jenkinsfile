pipeline {
    agent any
    
    tools {
        gradle 'gradle'
    }
    
    stages {
        stage('Test') {
            steps {
                sh './gradlew clean test'
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
                    sh './gradlew sonarqube'
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build') {
            steps {
                sh './gradlew assemble javadoc'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/libs/*.jar'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline réussi!'
        }
        failure {
            echo 'Pipeline échoué!'
        }
    }
}
