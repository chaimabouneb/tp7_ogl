pipeline {
    agent any
    
    tools {
        gradle 'gradle' // Gradle doit être configuré dans Jenkins (Global Tool Configuration)
    }
    
    environment {
        // Variables pour SonarQube (optionnel)
        SONAR_HOST_URL = 'http://localhost:9000'
        
        // Variables pour le déploiement (à remplacer)
        REPO_URL = 'https://mymavenrepo.com/repository/votre-repo/'
        CREDENTIALS_ID = 'mymavenrepo-creds'
    }
    
    stages {
        // ========== PHASE 1: TESTS ==========
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
                    
                    // Archivage des rapports HTML de test
                    publishHTML([
                        reportDir: 'build/reports/tests/test',
                        reportFiles: 'index.html',
                        reportName: 'Rapports de Tests Unitaires',
                        keepAll: true
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
                    withSonarQubeEnv('sonar') { // 'sonar' = nom configuré dans Jenkins
                        sh './gradlew sonarqube \
                            -Dsonar.projectKey=votre-projet \
                            -Dsonar.projectName="Votre Projet API" \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_AUTH_TOKEN' // Token stocké dans Jenkins
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
                    
                    // Publication du rapport Jacoco
                    publishHTML([
                        reportDir: 'build/reports/jacoco/test/html',
                        reportFiles: 'index.html',
                        reportName: 'Rapport de Couverture Jacoco',
                        keepAll: true
                    ])
                }
            }
        }
        
        // ========== PHASE 6: DÉPLOIEMENT ==========
        stage('Deploy') {
            when {
                expression {
                    // Ne déployer que sur la branche main/master
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
            
            // Nettoyage
            cleanWs()
        }
        
        success {
            echo '🎉 Toutes les étapes ont réussi !'
            
            // Notification par email
            emailext (
                to: 'team@example.com, dev@example.com',
                subject: "✅ SUCCÈS - Pipeline ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    Le pipeline CI/CD a été exécuté avec succès !
                    
                    DÉTAILS :
                    - Projet : ${env.JOB_NAME}
                    - Build : #${env.BUILD_NUMBER}
                    - Branche : ${env.BRANCH_NAME}
                    - Commit : ${env.GIT_COMMIT}
                    - URL du build : ${env.BUILD_URL}
                    
                    ARTEFACTS DISPONIBLES :
                    - JAR : ${env.BUILD_URL}artifact/
                    - Documentation : ${env.BUILD_URL}javadoc/
                    
                    L'API a été déployée avec succès sur MyMavenRepo.
                """,
                mimeType: 'text/html'
            )
            
            // Notification Slack (optionnel - nécessite plugin Slack)
            script {
                try {
                    slackSend(
                        channel: '#dev-notifications',
                        color: 'good',
                        message: """
                            ✅ *Déploiement Réussi* - ${env.JOB_NAME}
                            *Build*: #${env.BUILD_NUMBER}
                            *Branche*: ${env.BRANCH_NAME}
                            *État*: Déployé sur MyMavenRepo
                            *URL*: ${env.BUILD_URL}
                        """,
                        failOnError: false
                    )
                } catch (Exception e) {
                    echo "⚠️ Notification Slack échouée: ${e.message}"
                }
            }
        }
        
        failure {
            echo '❌ Le pipeline a échoué !'
            
            // Notification d'échec par email
            emailext (
                to: 'team@example.com, dev@example.com',
                subject: "❌ ÉCHEC - Pipeline ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    Le pipeline CI/CD a échoué !
                    
                    DÉTAILS :
                    - Projet : ${env.JOB_NAME}
                    - Build : #${env.BUILD_NUMBER}
                    - Branche : ${env.BRANCH_NAME}
                    - URL du build : ${env.BUILD_URL}
                    - Phase en échec : ${currentBuild.currentResult}
                    
                    Veuillez vérifier les logs pour plus de détails.
                """,
                mimeType: 'text/html'
            )
            
            // Notification Slack d'échec
            script {
                try {
                    slackSend(
                        channel: '#dev-notifications',
                        color: 'danger',
                        message: """
                            ❌ *Pipeline Échoué* - ${env.JOB_NAME}
                            *Build*: #${env.BUILD_NUMBER}
                            *Branche*: ${env.BRANCH_NAME}
                            *Cause*: ${currentBuild.currentResult}
                            *URL*: ${env.BUILD_URL}
                        """,
                        failOnError: false
                    )
                } catch (Exception e) {
                    echo "⚠️ Notification Slack échouée: ${e.message}"
                }
            }
        }
        
        unstable {
            echo '⚠️ Le pipeline est instable (tests échoués)'
            
            emailext (
                to: 'team@example.com',
                subject: "⚠️ INSTABLE - Pipeline ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: "Des tests ont échoué mais le build a continué.",
                mimeType: 'text/html'
            )
        }
        
        changed {
            echo '📈 Statut du pipeline changé depuis la dernière exécution'
        }
    }
}
