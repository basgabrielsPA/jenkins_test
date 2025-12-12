
pipeline {
  agent any

  environment {
    JMETER_TEST   = 'tests/loadtest.jmx'
    JMETER_IMAGE  = 'alpine/jmeter:5.6.3'
    RESULTS_DIR   = 'results'
    // Optional: narrow charts to key transactions
    SERIES_FILTER = '^(Login|Search|Checkout)$'
    // Optional: APDEX thresholds (ms)
    APDEX_SAT     = '500'
    APDEX_TOL     = '1500'
    // Optional: chart bucket size in ms
    GRANULARITY   = '60000'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Validate test plan') {
      steps {
        sh '''
          set -euxo pipefail
          echo "WORKSPACE: $WORKSPACE"
          test -r "$JMETER_TEST" || { echo "ERROR: $JMETER_TEST not found or not readable"; exit 1; }
        '''
      }
    }

    stage('Prepare JMeter user.properties') {
      steps {
        sh '''
          set -euxo pipefail

          # JMeter HTML report must be generated into a non-existing folder
          rm -rf "$RESULTS_DIR/report"
          mkdir -p "$RESULTS_DIR"

          # Create a dashboard-ready properties file
          cat > "$WORKSPACE/user.properties" <<EOF
# ---- SaveService: ensure all required CSV columns are present ----
jmeter.save.saveservice.output_format=csv
jmeter.save.saveservice.bytes=true
jmeter.save.saveservice.label=true
jmeter.save.saveservice.latency=true
jmeter.save.saveservice.response_code=true
jmeter.save.saveservice.response_message=true
jmeter.save.saveservice.successful=true
jmeter.save.saveservice.thread_counts=true
jmeter.save.saveservice.thread_name=true
jmeter.save.saveservice.time=true
jmeter.save.saveservice.connect_time=true
jmeter.save.saveservice.timestamp_format=yyyy/MM/dd HH:mm:ss

# ---- Dashboard tuning ----
jmeter.reportgenerator.apdex_satisfied_threshold=${APDEX_SAT}
jmeter.reportgenerator.apdex_tolerated_threshold=${APDEX_TOL}
aggregate_rpt_pct1=90
aggregate_rpt_pct2=95
aggregate_rpt_pct3=99
#jmeter.reportgenerator.exporter.html.series_filter=${SERIES_FILTER}
jmeter.reportgenerator.overall_granularity=${GRANULARITY}
EOF

          echo "Generated $WORKSPACE/user.properties:"
          sed -n '1,40p' "$WORKSPACE/user.properties" || true
        '''
      }
    }

    stage('Run JMeter (Docker, volumes-from jenkins)') {
      steps {
        sh '''
          set -euxo pipefail

          # Optional visibility check: the JMeter container should see the same workspace
          docker run --rm \
            --volumes-from jenkins \
            -u "$(id -u):$(id -g)" \
            -w "$WORKSPACE" \
            alpine:3.19 \
            sh -c 'ls -la "$PWD" && ls -la "$(dirname "$JMETER_TEST")"'

          # Run JMeter, write JTL, and generate HTML dashboard
          docker run --rm \
            --volumes-from jenkins \
            -u "$(id -u):$(id -g)" \
            -w "$WORKSPACE" \
            "$JMETER_IMAGE" \
            -n \
            -t "$JMETER_TEST" \
            -l "$RESULTS_DIR/results.jtl" \
            -f \
            -q "$WORKSPACE/user.properties" \
            -e -o "$RESULTS_DIR/report" \
            -j "$RESULTS_DIR/jmeter.log"
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
      // Handy for debugging dashboard generation issues
      archiveArtifacts artifacts: 'results/jmeter.log', allowEmptyArchive: true
    }
  }
}
