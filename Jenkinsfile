
pipeline {
  agent any

  environment {
    // Your test plan relative to the workspace
    JMETER_TEST = 'tests/loadtest.jmx'

    // Convenience variables
    RESULTS_DIR  = 'results'
    JMETER_IMAGE = 'alpine/jmeter:5.6.3'
    ALPINE_IMAGE = 'alpine:3.19'
    // Optionally: TZ = 'Europe/Amsterdam'
  }

  stages {
    stage('Checkout') {
      steps {
        // For "Pipeline script from SCM", this will pull from the configured repo
        checkout scm
      }
    }

    stage('Validate JMeter script') {
      steps {
        sh '''
          set -euxo pipefail
          echo "WORKSPACE: $WORKSPACE"
          ls -la
          ls -la "$(dirname "$JMETER_TEST")" || true

          # Ensure the test plan exists and is readable in the workspace
          test -r "$JMETER_TEST" || { echo "ERROR: $JMETER_TEST not readable"; exit 1; }
        '''
      }
    }

    stage('Run JMeter (Docker, named volume for results)') {
      steps {
        sh '''
          set -euxo pipefail

          # JMeter requires that the -o directory does not already exist
          rm -rf "$RESULTS_DIR/report"

          # Parent dir should exist in the workspace (we will copy into it later)
          mkdir -p "$RESULTS_DIR"

          # Create Docker named volume for reliable writes (safe if it already exists)
          docker volume create jmeter_results || true

          # Sanity-check: workspace mount visibility from a plain Alpine container
          docker run --rm \
            -u "$(id -u):$(id -g)" \
            -v "$WORKSPACE:/work" \
            -w /work \
            "$ALPINE_IMAGE" \
            ls -la /work >/dev/null

          # Run JMeter using absolute paths and matching UID/GID to avoid mount permission issues
          docker run --rm \
            -u "$(id -u):$(id -g)" \
            -v "$WORKSPACE:/work" \
            -v jmeter_results:/work/$RESULTS_DIR \
            -w /work \
            "$JMETER_IMAGE" \
            -n \
            -t "/work/$JMETER_TEST" \
            -l "/work/$RESULTS_DIR/results.jtl" \
            -f \
            -e -o "/work/$RESULTS_DIR/report" \
            -j "/work/$RESULTS_DIR/jmeter.log"

          # Copy results back from the named volume into the Jenkins workspace
          docker run --rm \
            -u "$(id -u):$(id -g)" \
            -v jmeter_results:/src \
            -v "$WORKSPACE:/dst" \
            "$ALPINE_IMAGE" \
            sh -c 'cp -r /src/* /dst/$RESULTS_DIR/'
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
}
