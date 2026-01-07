pipeline {
    agent any
    
    tools {
        gradle 'gradle'
    }
    
    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        REPO_URL = 'https://mymavenrepo.com/repository/votre-repo/'
        CREDENTIALS_ID = 'mymavenrepo-creds'
    }
    
    stages {
        
        stage('Test') {
            steps {
                script {
                    echo '🚀 Démarrage des tests unitaires...'
                    sh './gradlew clean test'
                }
            }
            post {
                always {
                    // Archivage des résultats JUnit
                    junit 'build/test-results/test/**/*.xml'
                    
                    // CORRECTION ICI: Ajout des paramètres requis
                    publishHTML([
                        reportDir: 'build/reports/tests/test',
                        reportFiles: 'index.html',
                        reportName: 'Rapports de Tests Unitaires',
                        keepAll: true,
                        alwaysLinkToLastBuild: false,
                        allowMissing: false
                    ])
                }
            }
        }
        
        // ========== PHASE 2: RAPPORTS CUCUMBER ==========
        stage('Cucumber Reports') {
            steps {
                script {
                    echo '📊 Génération des rapports Cucumber...'
                    cucumber buildStatus: 'UNSTABLE',
                            fileIncludePattern: '**/*.json',
                            jsonReportDirectory: 'build/cucumber-reports',
                            sortingMethod: 'ALPHABETICAL',
                            trendsLimit: 10
                }
            }
        }
        
        // ========== PHASE 3: ANALYSE DE CODE ==========
        stage('Code Analysis') {
            steps {
                script {
                    echo '🔍 Analyse du code avec SonarQube...'
                    withSonarQubeEnv('sonar') {
                        sh './gradlew sonarqube \
                            -Dsonar.projectKey=tp7-api \
                            -Dsonar.projectName="TP7 API" \
                            -Dsonar.host.url=$SONAR_HOST_URL'
                    }
                }
            }
        }
        
        // ========== PHASE 4: QUALITÉ DU CODE ==========
        stage('Code Quality') {
            steps {
                script {
                    echo '📈 Vérification de la Quality Gate...'
                    timeout(time: 10, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        
        // ========== PHASE 5: CONSTRUCTION ==========
        stage('Build') {
            steps {
                script {
                    echo '🏗️ Construction du projet...'
                    
                    // Génération du JAR
                    sh './gradlew assemble'
                    
                    // Génération de la documentation Javadoc
                    sh './gradlew javadoc'
                    
                    // Génération du rapport de couverture Jacoco
                    sh './gradlew jacocoTestReport'
                }
            }
            post {
                success {
                    // Archivage des artefacts
                    archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                    archiveArtifacts artifacts: 'build/docs/javadoc/**/*', fingerprint: true
                    
                    // CORRECTION ICI: Ajout des paramètres requis
                    publishHTML([
                        reportDir: 'build/reports/jacoco/test/html',
                        reportFiles: 'index.html',
                        reportName: 'Rapport de Couverture Jacoco',
                        keepAll: true,
                        alwaysLinkToLastBuild: false,
                        allowMissing: false
                    ])
                }
            }
        }
        
        // ========== PHASE 6: DÉPLOIEMENT ==========
        stage('Deploy') {
            when {
                expression {
                    return env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master'
                }
            }
            steps {
                script {
                    echo '🚚 Déploiement vers MyMavenRepo...'
                    
                    withCredentials([
                        usernamePassword(
                            credentialsId: env.CREDENTIALS_ID,
                            usernameVariable: 'REPO_USER',
                            passwordVariable: 'REPO_PASS'
                        )
                    ]) {
                        sh """
                            ./gradlew publish \
                                -PrepoUrl="${env.REPO_URL}" \
                                -PrepoUser="$REPO_USER" \
                                -PrepoPassword="$REPO_PASS" \
                                -Pversion=${env.BUILD_NUMBER}
                        """
                    }
                }
            }
        }
    }
    
    // ========== NOTIFICATIONS ==========
    post {
        always {
            echo "✅ Pipeline ${currentBuild.currentResult} - Build #${env.BUILD_NUMBER}"
        }
        
        success {
            echo '🎉 Toutes les étapes ont réussi !'
            
            // Notification par email simplifiée
            emailext (
                to: 'team@example.com',
                subject: "✅ SUCCÈS - Pipeline ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: "Le pipeline CI/CD a été exécuté avec succès!\n\nURL: ${env.BUILD_URL}"
            )
        }
        
        failure {
            echo '❌ Le pipeline a échoué !'
            
            emailext (
                to: 'team@example.com',
                subject: "❌ ÉCHEC - Pipeline ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: "Le pipeline CI/CD a échoué!\n\nURL: ${env.BUILD_URL}"
            )
        }
    }
}
