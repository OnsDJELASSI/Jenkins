pipeline {
    agent any

    environment {
        ZAP_PORT = '7075'
        ZAP_TARGET = 'http://juice-shop.herokuapp.com' 
        ZAP_CONTAINER_NAME = 'zap_juice_scan'
        SCAN_POLICY = 'Default Policy'
        WORKSPACE = "${env.WORKSPACE}"
    }

    stages {
        stage('Préparation') {
            steps {
                cleanWs()
                git url: 'https://github.com/OnsDJELASSI/Jenkins.git', branch: 'main'
            }
        }

        stage('Démarrage ZAP') {
            steps {
                script {
                    // Nettoyage des anciens conteneurs
                    sh "docker rm -f ${ZAP_CONTAINER_NAME} || true"
                    
                    // Lancement avec volume persistant
                    sh """
                    docker run -d --rm --name ${ZAP_CONTAINER_NAME} \
                      -v ${WORKSPACE}:/zap/wrk:rw \
                      -p ${ZAP_PORT}:${ZAP_PORT} \
                      owasp/zap2docker-stable:2.14.0 \
                      zap-x.sh -daemon -host 0.0.0.0 -port ${ZAP_PORT} \
                      -config api.addrs.addr.name=.* \
                      -config api.addrs.addr.regex=true \
                      -config api.disablekey=true \
                      -config database.newsession=3
                    """
                    
                    // Attente intelligente
                    timeout(time: 2, unit: 'MINUTES') {
                        waitUntil {
                            sh "curl -sSf http://localhost:${ZAP_PORT}/JSON/core/action/newSession > /dev/null"
                            return true
                        }
                    }
                }
            }
        }

        stage('Exécution du scan') {
            steps {
                script {
                    // Configuration du contexte
                    sh """
                    curl -sS -X POST "http://localhost:${ZAP_PORT}/JSON/context/action/includeInContext/" \
                      --data-urlencode "contextName=JuiceShop" \
                      --data-urlencode "regex=${ZAP_TARGET}.*"
                    """
                    
                    // Démarrage du scan
                    def scanId = sh(returnStdout: true, script: """
                        curl -sS -X POST "http://localhost:${ZAP_PORT}/JSON/ascan/action/scan/" \
                          --data-urlencode "url=${ZAP_TARGET}" \
                          --data "contextName=JuiceShop" \
                          --data "scanPolicyName=${SCAN_POLICY}" \
                          --data "recurse=true" \
                          --data "inScopeOnly=true" \
                          | jq -r '.scan'
                    """).trim()

                    // Surveillance du scan
                    while(true) {
                        def status = sh(returnStdout: true, script: """
                            curl -sS "http://localhost:${ZAP_PORT}/JSON/ascan/view/status/?scanId=${scanId}" \
                            | jq -r '.status'
                        """).trim()
                        
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
                    // Génération des rapports
                    sh """
                    docker exec ${ZAP_CONTAINER_NAME} zap-cli report \
                      -o /zap/wrk/zap_report.html \
                      -f html
                    
                    docker exec ${ZAP_CONTAINER_NAME} zap-cli report \
                      -o /zap/wrk/zap_report.json \
                      -f json
                    """
                    
                    // Vérification des résultats
                    def alerts = sh(returnStdout: true, script: """
                        jq '.alerts | length' zap_report.json
                    """).trim().toInteger()
                    
                    if(alerts > 0) {
                        unstable("Vulnérabilités détectées: ${alerts}")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Arrêt propre du conteneur
                sh "docker stop ${ZAP_CONTAINER_NAME} || true"
                
                // Archivage
                archiveArtifacts artifacts: 'zap_report.*', allowEmptyArchive: true
                
                // Publication HTML
                publishHTML(
                    target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'zap_report.html',
                        reportName: 'Rapport ZAP'
                    ]
                )
            }
        }
    }
}
