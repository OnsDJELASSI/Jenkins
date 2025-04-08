pipeline {
    agent any

    environment {
        ZAP_PORT = '7075'
        ZAP_TARGET = 'http://juice-shop.herokuapp.com' 
        ZAP_CONTAINER_NAME = 'zap_scan'
        SCAN_POLICY = 'Default Policy'
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
                    // Nettoyer les anciens conteneurs
                    sh "docker rm -f ${ZAP_CONTAINER_NAME} || true"
                    
                    // Lancer le conteneur ZAP
                    sh """
                    docker run -d --rm --name ${ZAP_CONTAINER_NAME} \
                      -p ${ZAP_PORT}:${ZAP_PORT} \
                      owasp/zap2docker-stable:2.14.0 \
                      zap.sh -daemon -host 0.0.0.0 -port ${ZAP_PORT} \
                      -config api.addrs.addr.name=.* \
                      -config api.addrs.addr.regex=true \
                      -config api.disablekey=true
                    """
                    
                    // Attendre l'initialisation
                    sleep 120
                }
            }
        }

        stage('Vérification API') {
            steps {
                script {
                    waitUntil {
                        try {
                            sh "curl -sSf http://localhost:${ZAP_PORT}/JSON/version"
                            return true
                        } catch (Exception e) {
                            sleep 10
                            return false
                        }
                    }
                }
            }
        }

        stage('Scan actif') {
            steps {
                script {
                    // Démarrer le scan
                    def scanId = sh(script: """
                        curl -sS -X POST "http://localhost:${ZAP_PORT}/JSON/ascan/action/scan/" \
                          --data-urlencode "url=${ZAP_TARGET}" \
                          --data "contextName=JuiceShop_Scan" \
                          --data "scanPolicyName=${SCAN_POLICY}" \
                          --data "recurse=true" \
                          | jq -r '.scan'
                    """, returnStdout: true).trim()

                    echo "Scan ID: ${scanId}"

                    // Surveiller la progression
                    def progress = "0"
                    while (progress != "100") {
                        sleep 30
                        progress = sh(script: """
                            curl -sS "http://localhost:${ZAP_PORT}/JSON/ascan/view/status/?scanId=${scanId}" \
                            | jq -r '.status'
                        """, returnStdout: true).trim()
                        echo "Progression du scan: ${progress}%"
                    }
                }
            }
        }

        stage('Génération rapport') {
            steps {
                script {
                    // Générer le rapport HTML et JSON
                    sh """
                        curl -sS "http://localhost:${ZAP_PORT}/OTHER/core/other/htmlreport/" -o zap_report.html
                        curl -sS "http://localhost:${ZAP_PORT}/OTHER/core/other/jsonreport/" -o zap_report.json
                    """
                }
            }
        }

        stage('Archivage') {
            steps {
                archiveArtifacts artifacts: 'zap_report.*', allowEmptyArchive: false
                zap scan: 'zap_report.html'
            }
        }
    }

    post {
        always {
            script {
                // Nettoyage des conteneurs
                sh "docker stop ${ZAP_CONTAINER_NAME} || true"
                sh "docker rm ${ZAP_CONTAINER_NAME} || true"
                
                // Publier les rapports
                junit testResults: 'zap_report.xml', allowEmptyResults: true
                publishHTML target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'zap_report.html',
                    reportName: 'Rapport ZAP'
                ]
            }
        }
    }
}
