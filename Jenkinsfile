pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Avsingh4141/8.2CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm install --no-audit --no-fund || exit /b 0'
      }
    }

    stage('Run Tests') {
      steps {
        bat 'npm test > test-output.txt 2>&1 || exit /b 0'
        archiveArtifacts artifacts: 'test-output.txt', allowEmptyArchive: true
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        bat 'npm audit --json > npm-audit.json || exit /b 0'
        archiveArtifacts artifacts: 'npm-audit.json', allowEmptyArchive: true
      }
    }
  }

  post {
    always {
      bat 'tar -czf pipeline-logs.tar.gz test-output.txt npm-audit.json || exit /b 0'
      archiveArtifacts artifacts: 'pipeline-logs.tar.gz', allowEmptyArchive: true
    }
  }
}
