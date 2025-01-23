pipeline {
    agent any
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv( 'sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=nodejsmysql -Dsonar.projectName=nodejsmysql"
                }
            }
        }
    }
}