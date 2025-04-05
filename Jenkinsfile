pipeline {
    agent any

    environment {
        ZAP_IMAGE = "owasp/zap2docker-stable"
        TARGET_URL = "http://testphp.vulnweb.com"
        REPORT_NAME = "zap_report.html"
    }

    stages {
        stage('Pull ZAP Docker Image') {
            steps {
                echo "Pulling ZAP Docker image..."
                sh "docker pull ${ZAP_IMAGE}"
            }
        }

        stage('Run ZAP Baseline Scan') {
            steps {
                echo "Running baseline scan on ${TARGET_URL}..."
                sh """
                    docker run --rm \
                        -v \$PWD:/zap/wrk \
                        ${ZAP_IMAGE} zap-baseline.py \
                        -t ${TARGET_URL} \
                        -r ${REPORT_NAME}
                """
            }
        }

        stage('Publish ZAP Report') {
            steps {
                echo "Publishing ZAP scan report..."
                archiveArtifacts artifacts: "${REPORT_NAME}", onlyIfSuccessful: true
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
    }
}
