
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/basgabrielsPA/jenkins_test.git'
            }
        }
        stage('Run JMeter Test') {
            steps {
                sh '''
                docker run --rm -v $PWD:/work -w /work justb4/jmeter:5.6.3 \
                -n -t tests/loadtest.jmx -l results/results.jtl -e -o results/report
                '''
            }
        }
        stage('Publish Report') {
            steps {
                publishHTML(target: [
                    reportDir: 'results/report',
                    reportFiles: 'index.html',
                    reportName: 'JMeter Report'
                ])
            }
        }
    }
}
