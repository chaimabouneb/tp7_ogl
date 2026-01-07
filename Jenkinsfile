pipeline {
    agent any
    tools {
        gradle 'gradle'
    }
    
    stages {
        stage('Test') {
            steps {
                sh './gradlew test'
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh './gradlew assemble'
            }
        }
    }
    
    post {
        failure {
            echo 'Pipeline a échoué!'
        }
        success {
            echo 'Pipeline réussi!'
        }
    }
}
