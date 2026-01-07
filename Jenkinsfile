pipeline {
    agent any
    
    tools {
        gradle 'gradle'
    }
    
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
        
       
}
