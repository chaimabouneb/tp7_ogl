pipeline {
    agent any

    environment {
        SONARQUBE_TOKEN = credentials('sonar-token')
        JAVA_HOME = tool name: 'JDK8', type: 'jdk' // Make sure JDK8 is installed in Jenkins
        PATH = "${tool 'Gradle8'}/bin:${env.PATH}" // Make sure Gradle 8 is installed
    }

    options {
        timestamps() // Add timestamps to logs
        buildDiscarder(logRotator(numToKeepStr: '10')) // Keep last 10 builds
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                script {
                    // Run Gradle online to download dependencies
                    bat './gradlew clean test jacocoTestReport'
                }
            }
        }

        stage('Publish Test Results') {
            steps {
                junit 'build/test-results/test/*.xml'
            }
        }

        stage('Code Coverage') {
            steps {
                jacoco(
                    execPattern: 'build/jacoco/test.exec',
                    classPattern: 'build/classes/java/main',
                    sourcePattern: 'src/main/java',
                    inclusionPattern: '**/*.class',
                    exclusionPattern: '**/*Test*'
                )
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    bat """
                        ./gradlew sonarqube \
                        -Dsonar.projectKey=tp7_ogl \
                        -Dsonar.host.url=http://YOUR_SONARQUBE_SERVER:9000 \
                        -Dsonar.login=${SONARQUBE_TOKEN} \
                        -Dsonar.java.source=1.8 \
                        -Dsonar.java.target=1.8 \
                        -Dsonar.junit.reportPaths=build/test-results/test \
                        -Dsonar.jacoco.reportPaths=build/jacoco/test.exec
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Check console output and reports."
        }
        success {
            echo "Build succeeded!"
        }
        failure {
            echo "Build failed!"
        }
    }
}
