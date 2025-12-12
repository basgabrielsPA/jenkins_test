
pipeline {
  agent any
  environment {
    JMETER_TEST = 'tests/loadtest.jmx'
  }

  stages {
    stage('Checkout') {
      steps {
        // If your job is "Pipeline script from SCM", prefer:
        checkout scm
        // Otherwise use:
        // git url: 'https://github.com/basgabrielsPA/jenkins_test', branch: 'main'
      }
    }

    stage('Validate JMeter script') {
      steps {
        sh '''
          echo "Workspace: $WORKSPACE"
          test -f "$JMETER_TEST" || { echo "ERROR: $JMETER_TEST not found"; ls -la; exit 1; }
        '''
      }
    }

    
	
	stage('Run JMeter via Docker') {
	  steps {
		sh '''
		  set -euxo pipefail

		  # Pre-checks in Jenkins workspace
		  echo "WORKSPACE: $WORKSPACE"
		  ls -la
		  ls -la tests || true
		  test -r "$JMETER_TEST"

		  # Ensure fresh report dir requirement
		  rm -rf results/report
		  mkdir -p results

		  # Create named volume (safe if it already exists)
		  docker volume create jmeter_results || true

		  # Run JMeter with absolute paths and matching UID/GID
		  docker run --rm \
			-u "$(id -u):$(id -g)" \
			-v "$WORKSPACE:/work" \
			-v jmeter_results:/work/results \
			-w /work \
			alpine/jmeter:5.6.3 \
			-n \
			-t "/work/$JMETER_TEST" \
			-l "/work/results/results.jtl" \
			-e -o "/work/results/report" \
			-j "/work/results/jmeter.log"

		  # Copy results back from the named volume into the workspace
		  docker run --rm \
			-u "$(id -u):$(id -g)" \
			-v jmeter_results:/src \
			-v "$WORKSPACE:/dst" \
			alpine:3.19 sh -c 'cp -r /src/* /dst/results/'
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
