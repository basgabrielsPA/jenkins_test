
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
		  rm -rf results/report
		  mkdir -p results

		  # Use a named volume for the /work/results path (Docker manages perms)
		  docker volume create jmeter_results || true

		  docker run --rm \
			-v "$PWD:/work" \
			-v jmeter_results:/work/results \
			-w /work \
			alpine/jmeter:5.6.3 \
			-n -t "$JMETER_TEST" \
			-l /work/results/results.jtl \
			-e -o /work/results/report

		  # Copy results back into the workspace so Jenkins can publish/archive them
		  docker run --rm \
			-v jmeter_results:/src \
			-v "$PWD:/dst" \
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
