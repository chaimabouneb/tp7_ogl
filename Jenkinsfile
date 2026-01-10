pipeline {
    agent any
    
    stages {
        stage('Test') {
            steps {
                bat './gradlew test'  // Changed from just './gradlew'
            }
            post {
                always {
                    junit 'build/test-results/test/**/*.xml'
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {  // Ensure 'sonar' matches your Jenkins config
                        // Run sonar task directly
                        bat './gradlew sonar'  // Simplified - no need for separate compile
                    }
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                bat './gradlew assemble javadoc'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**/*'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                bat './gradlew publish'
            }
        }
    }
}
