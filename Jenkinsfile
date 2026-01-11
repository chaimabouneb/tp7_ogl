node {
    // 1. Checkout the code
    stage('Checkout') {
        checkout scm
    }

    // 2. Build with Gradle (Targeting Java 11)
    stage('Build & Test') {
        bat './gradlew clean test jacocoTestReport'
    }

    // 3. Run Sonar Scanner CLI directly
    stage('SonarQube Analysis') {
        // This gets the path to the scanner you configured in Jenkins Tools
        def scannerHome = tool 'SonarScanner' 
        
        withSonarQubeEnv('sonar') {
            bat "${scannerHome}/bin/sonar-scanner " +
                "-Dsonar.projectKey=projet1_main " +
                "-Dsonar.sources=src/main/java " +
                "-Dsonar.java.binaries=build/classes/java/main " +
                "-Dsonar.java.source=11"
        }
    }
}
