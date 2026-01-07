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
     stages {
        // Phase 2.2: Code Analysis
        stage('Code Analysis') {
            steps {
                bat './gradlew sonarqube'
            }
        }
        
        // Phase 2.3: Code Quality
        stage('Code Quality') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        // Phase 2.4: Build
        stage('Build') {
            steps {
                // 1. Génération du fichier Jar
                // 2. Génération de la documentation
                bat './gradlew assemble javadoc'
            }
            post {
                success {
                    // 3. Archivage du fichier Jar et de la documentation
                    archiveArtifacts artifacts: 'build/libs/*.jar'
                    archiveArtifacts artifacts: 'build/docs/javadoc/**/*'
                }
            }
        }
    }
}
