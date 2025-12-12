
pipeline {
  agent any

  environment {
    JMETER_TEST = 'tests/loadtest.jmx'       // relative to $WORKSPACE
    JMETER_IMAGE = 'alpine/jmeter:5.6.3'
    ALPINE_IMAGE = 'alpine:3.19'
    RESULTS_DIR  = 'results'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Validate JMeter script (workspace)') {
      steps {
        sh '''
          set -euxo pipefail
          echo "WORKSPACE: $WORKSPACE"
          ls -la
          ls -la "$(dirname "$JMETER_TEST")" || true
          test -r "$JMETER_TEST" || { echo "ERROR: $JMETER_TEST not readable in workspace"; exit 1; }
        '''
      }
    }


stage('Run JMeter via Docker (mount Jenkins volumes)') {
  steps {
    sh '''
      set -euxo pipefail

      rm -rf results/report
      mkdir -p results

      # Visibility check
      docker run --rm \
        --volumes-from jenkins \
        -u "$(id -u):$(id -g)" \
        -w "$WORKSPACE" \
        alpine:3.19 sh -c 'ls -la "$WORKSPACE" && ls -la "$(dirname "$JMETER_TEST")"'

      # Run JMeter with explicit "save.saveservice" and dashboard properties
      docker run --rm \
        --volumes-from jenkins \
        -u "$(id -u):$(id -g)" \
        -w "$WORKSPACE" \
        alpine/jmeter:5.6.3 \
        -n \
        -t "$JMETER_TEST" \
        -l "results/results.jtl" -f \
        -Jjmeter.save.saveservice.output_format=csv \
        -Jjmeter.save.saveservice.bytes=true \
        -Jjmeter.save.saveservice.label=true \
        -Jjmeter.save.saveservice.latency=true \
        -Jjmeter.save.saveservice.response_code=true \
        -Jjmeter.save.saveservice.response_message=true \
        -Jjmeter.save.saveservice.successful=true \
        -Jjmeter.save.saveservice.thread_counts=true \
        -Jjmeter.save.saveservice.thread_name=true \
        -Jjmeter.save.saveservice.time=true \
        -Jjmeter.save.saveservice.connect_time=true \
        -Jjmeter.save.saveservice.timestamp_format="yyyy/MM/dd HH:mm:ss" \
        -Jjmeter.reportgenerator.apdex_satisfied_threshold=500 \
        -Jjmeter.reportgenerator.apdex_tolerated_threshold=1500 \
        -Jaggregate_rpt_pct1=90 -Jaggregate_rpt_pct2=95 -Jaggregate_rpt_pct3=99 \
        -Jjmeter.reportgenerator.exporter.html.series_filter="^(Login|Search|Checkout)$" \
        -Jjmeter.reportgenerator.overall_granularity=60000 \
        -e -o "results/report" \
        -j "results/jmeter.log"
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

  // Optional: always collect JMeter log even if the stage fails
  post {
    always {
      archiveArtifacts artifacts: 'results/jmeter.log', allowEmptyArchive: true
    }
  }
}
