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
        
        stage('Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        bat '''
                            ./gradlew clean compileJava compileTestJava
                            ./gradlew sonarqube \
                                -Dsonar.java.source=11 \
                                -Dsonar.java.target=11 \
                                -Dsonar.gradle.skipCompile=true
                        '''
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
