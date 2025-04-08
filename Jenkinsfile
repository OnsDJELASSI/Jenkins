pipeline {
    agent any

    environment {
        ZAP_PORT = '7075'
        ZAP_TARGET = 'http://juice-shop.herokuapp.com'
        ZAP_CONTAINER_NAME = "zap_juice_scan_${BUILD_NUMBER}"
        WORKSPACE = "${env.WORKSPACE}"
    }

    stages {
        stage('Préparation') {
            steps {
                cleanWs()
                git url: 'https://github.com/OnsDJELASSI/Jenkins.git', branch: 'main'
            }
        }

        stage('Démarrer ZAP') {
            steps {
                script {
                    sh "docker rm -f ${ZAP_CONTAINER_NAME} || true"
                    sh """
                    docker run -d --name ${ZAP_CONTAINER_NAME} \
                        -p ${ZAP_PORT}:${ZAP_PORT} \
                        -v ${WORKSPACE}:/zap/wrk:rw \
                        owasp/zap2docker-stable:2.14.0 \
                        zap-x.sh -daemon -host 0.0.0.0 -port ${ZAP_PORT} \
                        -config api.disablekey=true \
                        -config database.newsession=3
                    """
                    timeout(time: 2, unit: 'MINUTES') {
                        waitUntil {
                            sh "curl -sSf http://localhost:${ZAP_PORT}/JSON/core/view/version > /dev/null"
                            return true
                        }
                    }
                }
            }
        }

        stage('Configurer le contexte') {
            steps {
                script {
                    sh """
                    curl -sS -X POST "http://localhost:${ZAP_PORT}/JSON/context/action/newContext/" \
                        -d "contextName=JuiceShop"
                    """
                    sh """
                    curl -sS -X POST "http://localhost:${ZAP_PORT}/JSON/context/action/includeInContext/" \
                        --data-urlencode "contextName=JuiceShop" \
                        --data-urlencode "regex=${ZAP_TARGET}.*"
                    """
                }
            }
        }

        stage('Exécuter le scan') {
            steps {
                script {
                    sh """
                    curl -X POST "http://localhost:${ZAP_PORT}/JSON/ascan/action/scan/" \
                        --data-urlencode "url=${ZAP_TARGET}" \
                        -d "contextName=JuiceShop" \
                        -d "scanPolicyName=Default Policy" \
                        -d "recurse=true" \
                        -d "inScopeOnly=true"
                    """
                    
                    def scanId = sh(script: """
                        curl -s http://localhost:${ZAP_PORT}/JSON/ascan/view/scans/ \
                        | jq -r '.scans[-1].id'
                    """, returnStdout: true).trim()
                    
                    while(true) {
                        def status = sh(script: """
                            curl -s "http://localhost:${ZAP_PORT}/JSON/ascan/view/status/?scanId=${scanId}" \
                            | jq -r '.status'
                        """, returnStdout: true).trim()
                        
                        if (status == '100') break
                        echo "Progression: ${status}%"
                        sleep 30
                    }
                }
            }
        }

        stage('Générer rapports') {
            steps {
                script {
                    sh """
                    docker exec ${ZAP_CONTAINER_NAME} zap-cli report \
                        -o /zap/wrk/zap_report.html \
                        -f html
                        
                    docker exec ${ZAP_CONTAINER_NAME} zap-cli report \
                        -o /zap/wrk/zap_report.json \
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
            publishHTML target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '',
                reportFiles: 'zap_report.html',
                reportName: 'Rapport ZAP'
            ]
        }
    }
}
