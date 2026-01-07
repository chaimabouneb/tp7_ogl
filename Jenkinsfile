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
    }
}
