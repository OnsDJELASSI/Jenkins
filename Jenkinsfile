pipeline {
    agent any

    stages {
        stage('Start ZAP') {
            steps {
                script {
                    echo "Vérification si ZAP est déjà en cours d'exécution..."
                    def zapRunning = sh(script: "docker ps -q --filter name=zap", returnStdout: true).trim()

                    if (zapRunning == "") {
                        echo "Lancement du container ZAP..."
                        sh """
                            docker run -u zap -d -p 7075:7075 --name zap \
                            zaproxy/zap-stable:2.14.0 \
                            zap.sh -daemon -host 0.0.0.0 -port 7075 \
                            -config api.addrs.addr.name='.*' \
                            -config api.addrs.addr.regex=true \
                            -config api.disablekey=true
                        """
                        echo "Attente du démarrage complet de ZAP (10s)..."
                        sleep 10
                    } else {
                        echo "ZAP est déjà en cours d'exécution."
                    }
                }
            }
        }

        stage('Scan avec ZAP') {
            steps {
                script {
                    echo "Lancement du scan OWASP ZAP..."

                    // Remplacer example.com par un vrai site vulnérable
                    def targetUrl = "http://testphp.vulnweb.com"

                    def response = sh (
                        script: """curl -s -X GET "http://localhost:7075/JSON/ascan/action/scan" \\
                            -d "url=${targetUrl}" \\
                            -d "recurse=true" \\
                            -d "inContext=false"
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Réponse ZAP : ${response}"

                    if (response == "" || response.contains("Error")) {
                        error("ZAP n'a pas répondu correctement. Vérifie le container ou la cible.")
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Nettoyage du container ZAP..."
            sh "docker stop zap || true"
            sh "docker rm zap || true"
        }
    }
}
