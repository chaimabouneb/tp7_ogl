pipeline {
    agent any
    
    stages {
        stage('Test') {
            steps {
                bat './gradlew clean test'
            }
            post {
                always {
                    junit 'build/test-results/test/**/*.xml'
                }
            }
        }
     
        // Phase 2.2: Code Analysis - AVEC withSonarQubeEnv
        stage('Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {  // 'sonar' = nom configuré dans Jenkins
                        bat './gradlew sonar'
                    }
                }
            }
        }
        
        // Phase 2.3: Code Quality
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
