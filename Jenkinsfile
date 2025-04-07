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

        stage('Start ZAP') {
            steps {
                script {
                    def zapRunning = sh(script: "docker ps -q --filter name=zap", returnStdout: true).trim()
                    if (zapRunning) {
                        echo "ZAP est déjà en cours d'exécution."
                    } else {
                        echo "Démarrage de ZAP..."
                        sh '''
                        docker run -u zap -d -p ${ZAP_PORT}:${ZAP_PORT} --name zap \
                          zaproxy/zap-stable:2.14.0 \
                          zap.sh -daemon -host 0.0.0.0 -port ${ZAP_PORT} \
                          -config api.addrs.addr.name='.*' \
                          -config api.addrs.addr.regex=true \
                          -config api.disablekey=true
                        '''
                        sleep 30 // attendre que ZAP soit prêt
                    }
                }
            }
        }

        stage('Lancer le scan ZAP') {
            steps {
                echo "Lancement du scan ZAP..."
                sh """
                curl "http://localhost:${ZAP_PORT}/JSON/ascan/action/scan/?url=${ZAP_TARGET}&recurse=true"
                sleep 60
                """
            }
        }

        stage('Générer le rapport') {
            steps {
                echo "Exportation du rapport JSON..."
                sh """
                curl "http://localhost:${ZAP_PORT}/OTHER/core/other/jsonreport/" -o zap_report.json
                """
            }
        }

        stage('Arrêter ZAP') {
            steps {
                echo "Arrêt de ZAP..."
                sh "docker stop zap || true"
            }
        }
    }

    post {
        always {
            echo "Nettoyage..."
            sh "docker rm zap || true"
            archiveArtifacts artifacts: 'zap_report.json', allowEmptyArchive: true
        }
    }
}
