
pipeline {
  agent any
  environment {
    JMETER_TEST = 'tests/loadtest.jmx'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Run JMeter via Docker') {
      steps {
        sh '''
          set -e
          mkdir -p results
          docker run --rm \
            -v "$PWD:/work" \
            -w /work \
            justb4/jmeter:5.6.3 \
            -n -t "$JMETER_TEST" \
            -l results/results.jtl \
            -e -o results/report
        '''
      }
    }
    stage('Publish HTML Report') {
      steps {
        publishHTML(target: [
          reportDir: 'results/report',
          reportFiles: 'index.html',
          reportName: 'JMeter HTML Report',
          alwaysLinkToLastBuild: true,
          keepAll: true
        ])
      }
    }
    stage('Archive Results') {
      steps { archiveArtifacts artifacts: 'results/**', fingerprint: true }
    }
  }
}
