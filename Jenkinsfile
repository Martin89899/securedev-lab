pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Run Vulnerable App') {
            steps {
                sh '''
                    docker rm -f securedev-target || true
                    docker build -t securedev-app .
                    docker run -d --name securedev-target -p 8000:8000 securedev-app
                    sleep 5
                '''
            }
        }

        stage('OWASP ZAP Baseline Scan') {
            steps {
                sh '''
                    docker rm -f zap-scanner || true
                    docker run --name zap-scanner -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://host.docker.internal:8000 -r zap_report_1.html || true
                    docker cp zap-scanner:/zap/wrk/zap_report_1.html ./zap_report_1.html
                    docker rm -f zap-scanner || true
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'zap_report_1.html', allowEmptyArchive: true
            sh 'docker rm -f securedev-target || true'
        }
    }
}