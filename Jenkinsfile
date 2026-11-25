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
                    docker volume rm zap_reports || true
                    docker volume create zap_reports
                    docker run --rm -u root -v zap_reports:/zap/wrk/:rw -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://host.docker.internal:8000 -r zap_report_1.html || true
                    docker run --rm -v zap_reports:/src -v devsecops-tools_jenkins_home:/dst alpine cp /src/zap_report_1.html /dst/workspace/securedev-pipeline/zap_report_1.html
                    docker volume rm zap_reports || true
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