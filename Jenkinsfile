
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
		  rm -rf results/report         # ensure report dir does NOT exist
		  mkdir -p results              # parent must exist

		  # Use same uid/gid inside the JMeter container as the current Jenkins user
		  docker run --rm \
			-u "$(id -u):$(id -g)" \
			-v "$PWD:/work" \
			-w /work \
			alpine/jmeter:5.6.3 \
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
      steps {
        archiveArtifacts artifacts: 'results/**', fingerprint: true
      }
    }
  }
}
