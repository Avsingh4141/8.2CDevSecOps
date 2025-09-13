pipeline {
  agent any
  /* Polling triggers (if you don't use webhooks):
     Poll every 5 minutes: H/5 * * * *  */
  triggers { pollSCM('H/5 * * * *') }

  environment {
    // Set these in Jenkins job or leave empty: ENABLE_SONAR, ENABLE_EMAIL
    // ENABLE_SONAR = 'true' to run Sonar stage (requires SONAR_TOKEN credential)
    // ENABLE_EMAIL = 'true' to send email (requires Email Extension plugin and SMTP configured)
  }

  stages {
    stage('Checkout') {
      steps {
        // Update the URL below with your repo
        git branch: 'main', url: 'https://github.com/your_github_username/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test 2>&1 | tee test.log || true'
        archiveArtifacts artifacts: 'test.log', allowEmptyArchive: true
      }
    }

    stage('Coverage') {
      steps {
        sh 'npm run coverage 2>&1 | tee coverage.log || true'
        archiveArtifacts artifacts: 'coverage/lcov.info,coverage.log', allowEmptyArchive: true
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh 'npm audit --json > audit.json || true'
        sh 'npm audit 2>&1 | tee audit.log || true'
        archiveArtifacts artifacts: 'audit.json,audit.log', allowEmptyArchive: true
      }
    }

    stage('SonarCloud Analysis') {
      when { expression { return env.ENABLE_SONAR == 'true' } }
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh '''
            if [ ! -d sonar-scanner-cli ]; then
              wget -q https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip
              unzip -q sonar-scanner-cli-4.8.0.2856-linux.zip
              mv sonar-scanner-4.8.0.2856-linux sonar-scanner-cli
            fi
            export PATH=$PWD/sonar-scanner-cli/bin:$PATH
            sonar-scanner \
              -Dsonar.projectKey=YOUR_PROJECT_KEY \
              -Dsonar.organization=YOUR_ORGANIZATION \
              -Dsonar.host.url=https://sonarcloud.io \
              -Dsonar.login=$SONAR_TOKEN \
              -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info || true
          '''
        }
      }
    }

    stage('Send Email Notification') {
      when { expression { return env.ENABLE_EMAIL == 'true' } }
      steps {
        emailext subject: "Build ${currentBuild.fullDisplayName} - ${currentBuild.currentResult}",
                body: """Build URL: ${env.BUILD_URL}\nStatus: ${currentBuild.currentResult}""",
                to: 'your_email@example.com',
                attachLog: true,
                attachmentsPattern: 'test.log,audit.log'
      }
    }
  }

  post {
    always {
      echo "Archiving artifacts and finishing build"
    }
  }
}
