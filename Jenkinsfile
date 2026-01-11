node {
    // 1. Checkout the code
    stage('Checkout') {
        checkout scm
    }

    // 2. Build with Gradle (Targeting Java 11)
    stage('Build & Test') {
        bat './gradlew clean test jacocoTestReport'
    }

   stage('SonarQube Analysis') {
    def scannerHome = tool 'SonarScanner' 
    
    withSonarQubeEnv('sonar') {
        // We add SONAR_SCANNER_OPTS to bypass the Java security restrictions
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
