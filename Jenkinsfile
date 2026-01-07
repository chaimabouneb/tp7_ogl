pipeline {
    agent any
    
    stages {
        stage('Test') {
            steps {
                bat './gradlew clean test'
            }
        }
    }
}
