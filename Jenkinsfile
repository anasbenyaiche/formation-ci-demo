pipeline {
  agent any
  tools { maven 'Maven' }
  triggers { pollSCM('H/2 * * * *') }
  environment { NEXUS_URL = 'http://nexus:8081' }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build & Tests') {
      steps {
        // TODO A : construire et tester en mode batch
        sh 'mvn -B clean verify'
      }
    }
    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar -Dsonar.projectKey=demo-ci'
        }
      }
    }
    stage('Quality Gate') {
    steps {
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
    }
    }
    stage('Publish') {
      steps {
        // TODO B : injecter le credential 'nexus-deploy'
        //          dans NEXUS_USER / NEXUS_PASSWORD, puis pub lier
        //          avec ci-settings.xml vers $NEXUS_URL
        withCredentials([usernamePassword(credentialsId: 'nexus-deploy',
        usernameVariable: 'NEXUS_USER',
                         passwordVariable: 'NEXUS_PASSWORD')]) {
          sh 'mvn -B -s ci-settings.xml -DskipTests -Dnexus.url=$NEXUS_URL deploy'
        }
      }
    }
  }

  post {
      // TODO C : toujours publier les rapports de tests
      // TODO D : en cas de succès, archiver le JAR
      always  { junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml' }
    success { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
  }
}
