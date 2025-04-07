pipeline {
    agent any

    environment {
        ZAP_PORT = '7075'
        ZAP_TARGET = 'https://testphp.vulnweb.com'
    }

    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git 'https://github.com/OnsDJELASSI/Jenkins.git'
            }
        }

        stage('Start ZAP in daemon mode') {
            steps {
                script {
                    def zapRunning = sh(script: "docker ps -q -f name=zap", returnStdout: true).trim()
                    if (zapRunning) {
                        echo "ZAP is already running."
                    } else {
                        echo "Starting ZAP container..."
                        sh '''
                        docker run -u zap -d -p ${ZAP_PORT}:${ZAP_PORT} --name zap \
                            zaproxy/zap-stable:2.14.0 \
                            zap.sh -daemon -host 0.0.0.0 -port ${ZAP_PORT} \
                            -config api.addrs.addr.name='.*' \
                            -config api.addrs.addr.regex=true \
                            -config api.disablekey=true
                        '''
                        sleep 30 // attendre le démarrage complet
                    }
                }
            }
        }

        stage('Run ZAP Scan') {
            steps {
                script {
                    echo "Launching active scan..."
                    sh """
                    curl "http://localhost:${ZAP_PORT}/JSON/ascan/action/scan/?url=${ZAP_TARGET}&recurse=true"
                    sleep 60
                    """
                }
            }
        }

        stage('Generate report') {
            steps {
                echo "Exporting scan report..."
                sh """
                curl "http://localhost:${ZAP_PORT}/OTHER/core/other/jsonreport/" -o zap_report.json
                """
            }
        }

        stage('Stop ZAP') {
            steps {
                echo "Stopping ZAP..."
                sh "docker stop zap || true"
            }
        }
    }

    post {
        always {
            echo "Cleanup and archive results..."
            sh "docker rm zap || true"
            archiveArtifacts artifacts: 'zap_report.json', allowEmptyArchive: true
        }
    }
}
