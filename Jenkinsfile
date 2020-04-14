pipeline {
    agent {
        docker {
            image 'node:6-alpine'
            args '-p 3000:3000 -p 5000:5000'
        }
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
              withSonarQubeEnv('Sonar') { 
                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:4.3.0.2102:sonar ' + 
                '-f all/pom.xml ' +
                '-Dsonar.projectKey=sample-react ' +
                '-Dsonar.login=$SONAR_UN ' +
                '-Dsonar.password=$SONAR_PW ' +
              }
              withSonarQubeEnv('My SonarQube Server') {
                 sh './jenkins/scripts/test.sh'
              }
                
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 3, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        stage('Deliver for development') {
            when {
                branch 'develop' 
            }
            steps {
                sh './jenkins/scripts/deliver-for-development.sh'                
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
        stage('Deploy for production') {
            when {
                branch 'master'  
            }
            steps {
                sh './jenkins/scripts/deploy-for-production.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}