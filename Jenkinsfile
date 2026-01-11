node {
    def javaHome = tool 'Java11'
    
    // This sets the environment variables for this specific execution
    withEnv(["JAVA_HOME=${javaHome}", "PATH+JAVA=${javaHome}/bin"]) {
        
        stage('Checkout') {
            checkout scm
        }

        stage('Build & Test') {
            // Now ./gradlew will use Java 11 automatically
            bat './gradlew clean test jacocoTestReport'
        }

        stage('SonarQube Analysis') {
            def scannerHome = tool 'SonarScanner'
            withSonarQubeEnv('sonar') {
                // We keep the --add-opens just in case, but Java 11 is much more stable for Sonar 7.4
                withEnv(["SONAR_SCANNER_OPTS=--add-opens java.base/java.lang=ALL-UNNAMED"]) {
                    bat "${scannerHome}/bin/sonar-scanner " +
                        "-Dsonar.projectKey=projet1_main " +
                        "-Dsonar.sources=src/main/java " +
                        "-Dsonar.java.binaries=build/classes/java/main " +
                        "-Dsonar.java.source=11"
                }
            }
        }
    }
}
