pipeline {
    agent any

    environment {
        ZAP_PORT = '7075'
        ZAP_TARGET = 'http://juice-shop.herokuapp.com'
        ZAP_CONTAINER_NAME = 'zap_juice_scan'
        WORKSPACE = "${env.WORKSPACE}"
    }

    stages {
        stage('Préparation') {
            steps {
                cleanWs()
                git 'https://github.com/OnsDJELASSI/Jenkins.git'
            }
        }

        stage('Démarrer ZAP') {
            steps {
                script {
                    sh "docker rm -f ${ZAP_CONTAINER_NAME} || true"
                    sh """
                    docker run -d --name ${ZAP_CONTAINER_NAME} \\
                      -p ${ZAP_PORT}:${ZAP_PORT} \\
                      -v ${WORKSPACE}:/zap/wrk \\
                      owasp/zap2docker-stable:2.14.0 \\
                      zap-x.sh -daemon -host 0.0.0.0 -port ${ZAP_PORT} \\
                      -config api.disablekey=true
                    """
                    sleep 60
                }
            }
        }

        stage('Scan actif') {
            steps {
                script {
                    sh """
                    curl -X POST "http://localhost:${ZAP_PORT}/JSON/ascan/action/scan/" \\
                      -d "url=${ZAP_TARGET}" \\
                      -d "recurse=true" \\
                      -d "inScopeOnly=true" \\
                      -d "scanPolicyName=Default Policy"
                    """
                    
                    def scanId = sh(script: """
                        curl -s http://localhost:${ZAP_PORT}/JSON/ascan/view/scans/ \\
                        | jq -r '.scans[0].id'
                    """, returnStdout: true).trim()
                    
                    while(true) {
                        def status = sh(script: """
                            curl -s "http://localhost:${ZAP_PORT}/JSON/ascan/view/status/?scanId=${scanId}" \\
                            | jq -r '.status'
                        """, returnStdout: true).trim()
                        
                        if (status == '100') break
                        echo "Progression: ${status}%"
                        sleep 30
                    }
                }
            }
        }

        stage('Rapports') {
            steps {
                script {
                    sh """
                    docker exec ${ZAP_CONTAINER_NAME} zap-cli report \\
                      -o /zap/wrk/zap_report.html \\
                      -f html
                    
                    docker exec ${ZAP_CONTAINER_NAME} zap-cli report \\
                      -o /zap/wrk/zap_report.json \\
                      -f json
                    """
                }
            }
        }

        stage('Nettoyage') {
            steps {
                script {
                    sh "docker stop ${ZAP_CONTAINER_NAME} || true"
                    sh "docker rm ${ZAP_CONTAINER_NAME} || true"
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'zap_report.*', allowEmptyArchive: true
        }
    }
}
