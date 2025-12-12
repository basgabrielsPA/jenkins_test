
pipeline {
    agent any

    environment {
        JMETER_TEST = 'tests/loadtest.jmx'
    }

    stages {
        stage('Checkout') {
            steps {
                // Jenkins will do this automatically if using "Pipeline from SCM"
                checkout scm
            }
        }

        stage('Validate JMeter Script') {
            steps {
                sh '''
                  echo "Workspace: $WORKSPACE"
                  if [ ! -f "$JMETER_TEST" ]; then
                    echo "ERROR: JMeter script not found at $JMETER_TEST"
                    exit 1
                  fi
                '''
            }
        }

        stage('Run JMeter Test') {
            steps {
                sh '''
                  mkdir -p results
                  # Run JMeter in non-GUI mode
                  jmeter -n -t "$JMETER_TEST" -l results/results.jtl -e -o results/report
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
            steps {
                archiveArtifacts artifacts: 'results/**', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Results saved in $WORKSPACE/results"
        }
        failure {
            echo "Build failed. Check Console Output and HTML report."
        }
    }
}
