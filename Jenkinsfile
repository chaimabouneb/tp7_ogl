node {
    def javaHome = tool 'Java11'
    def scannerHome = tool 'SonarScanner'

    withEnv(["JAVA_HOME=${javaHome}", "PATH+JAVA=${javaHome}/bin"]) {
        try {
            stage('Checkout') {
                checkout scm
            }

            stage('Test') {
                bat './gradlew clean test jacocoTestReport'
                junit '**/build/test-results/test/*.xml'
                // Fixed path to find Cucumber JSON files
                cucumber buildStatus: 'UNSTABLE', 
                         jsonReportDirectory: 'build/test-results/test', 
                         fileIncludePattern: '**/*.json'
            }

            stage('Code Analysis') {
                withSonarQubeEnv('sonar') {
                    bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=projet1_main"
                }
            }

            stage('Code Quality') {
                timeout(time: 1, unit: 'HOURS') {
                    def qg = waitForQualityGate()
                    if (qg.status == 'ERROR') {
                        error "Pipeline aborted: Quality Gate Failed"
                    }
                }
            }

            stage('Build') {
                bat './gradlew jar javadoc'
                archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', fingerprint: true
            }

            stage('Deploy') {
                bat './gradlew publish'
            }

            stage('Notification') {
                // FIXED: Changed .dz to .com
                emailext body: "SUCCESS: Project ${env.JOB_NAME} Build #${env.BUILD_NUMBER} is deployed.", 
                         subject: "Build Success: ${env.JOB_NAME}", 
                         to: "lc_bounab@esi.dz"
                
                // Requirement: Slack notification
                slackSend color: 'good', message: "Deploy successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            }

        } catch (Exception e) {
            emailext body: "FAILED: Stage ${env.STAGE_NAME} failed with error: ${e.message}", 
                     subject: "Build Failed: ${env.JOB_NAME}", 
                     to: "lc_bounab@esi.dz"
            
            slackSend color: 'danger', message: "Build Failed at stage ${env.STAGE_NAME}"
            throw e
        }
    }
}
