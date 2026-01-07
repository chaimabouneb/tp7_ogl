pipeline {
    agent any
    
    stages {
        stage('Test') {
            steps {
                bat './gradlew'
            }
            post {
                always {
                    junit 'build/test-results/test/**/*.xml'
                }
            }
        }
     
        // Phase 2.2: Code Analysis
        stage('Code Analysis') {
            steps {
                bat './gradlew sonarqube'
            }
        }
        
        
        
           // Phase 2.3: Code quality
        stage('Code Quality') {
            steps {
                  waitForQualityGate abortPipeline: true 
            }
        }
        
            }

}
