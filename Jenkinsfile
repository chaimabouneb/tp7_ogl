node {
   
    def javaHome = tool 'Java11'
    def scannerHome = tool 'SonarScanner'

    withEnv(["JAVA_HOME=${javaHome}", "PATH+JAVA=${javaHome}/bin"]) {
        try {
            stage('Checkout') {
                checkout scm
            }

            stage('Test') {
                // 1. Launch unit tests [cite: 24]
                bat './gradlew clean test jacocoTestReport'
                
                // 2. Archive unit test results [cite: 25]
                junit '**/build/test-results/test/*.xml'
                
                // 3. Generate Cucumber reports [cite: 26, 59]
                cucumber buildStatus: 'UNSTABLE', 
                         jsonReportDirectory: 'build/test-results/test', 
                         fileIncludePattern: '**/cucumber.json'
            }

            stage('Code Analysis') {
                // Analyze quality with SonarQube [cite: 28]
                withSonarQubeEnv('sonar') {
                    bat "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=projet1_main"
                }
            }

            stage('Code Quality') {
                // Verify Quality Gate status; stop if Failed [cite: 30, 31]
                timeout(time: 1, unit: 'HOURS') {
                    def qg = waitForQualityGate()
                   if (qg.status == 'ERROR') {
                        error "Pipeline aborted: Quality Gate Failed (${qg.status})"
                    }
                }
            }

            stage('Build') {
                // 1. Generate Jar and 2. Documentation [cite: 34, 35]
                bat './gradlew jar javadoc'
                
                // 3. Archive Jar and documentation [cite: 36]
                archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', 
                                 fingerprint: true
            }

            stage('Deploy') {
                // Deploy Jar to MyMavenRepo [cite: 38, 41]
                bat './gradlew publish'
            }

            stage('Notification') {
                // Success: Email and Slack [cite: 42]
                emailext body: "Deployment Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}", 
                         subject: "Success: ${env.JOB_NAME}", 
                         to: "chaimabouneb19@gmail.dz"
                
                // Note: Requires Slack plugin and configured workspace
                // slackSend color: 'good', message: "Successful deployment of ${env.JOB_NAME}"
            }

        } catch (Exception e) {
            // Notify team on failure in any phase [cite: 43]
            emailext body: "Pipeline Failed at stage ${env.STAGE_NAME}. Error: ${e.message}", 
                     subject: "FAILED: ${env.JOB_NAME}", 
                     to: "chaimabouneb19@gmail.com"
            throw e
        }
    }
}
