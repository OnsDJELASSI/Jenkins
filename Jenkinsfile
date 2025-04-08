pipeline {
    agent any

    environment {
        ZAP_PORT = '7075' // Port de l'API ZAP
        ZAP_TARGET = 'https://testphp.vulnweb.com' // URL cible à scanner
        ZAP_CONTAINER_NAME = 'zap' // Nom du conteneur ZAP
    }

    stages {
        stage('Clean workspace') {
            steps {
                cleanWs() // Nettoyage du workspace avant de commencer
            }
        }

        stage('Git Checkout') {
            steps {
                git 'https://github.com/OnsDJELASSI/Jenkins.git' // Clonage du dépôt Git
            }
        }

        stage('Start ZAP') {
            steps {
                script {
                    // Vérification si ZAP est déjà en cours d'exécution
                    def zapRunning = sh(script: "docker ps -q --filter name=${ZAP_CONTAINER_NAME}", returnStdout: true).trim()
                    if (zapRunning) {
                        echo "ZAP est déjà en cours d'exécution."
                    } else {
                        echo "Démarrage de ZAP..."
                        sh """
                        docker run -d --name ${ZAP_CONTAINER_NAME} -p ${ZAP_PORT}:${ZAP_PORT} \
                          zaproxy/zap-stable:2.14.0 \
                          zap.sh -daemon -host 0.0.0.0 -port ${ZAP_PORT} \
                          -config api.addrs.addr.name='.*' \
                          -config api.addrs.addr.regex=true \
                          -config api.disablekey=true
                        """
                        sleep 30 // Attendre que ZAP soit prêt
                    }
                }
            }
        }

        stage('Lancer le scan ZAP') {
            steps {
                echo "Lancement du scan ZAP..."
                sh """
                curl -X GET "http://localhost:${ZAP_PORT}/JSON/ascan/action/scan" \
                    -d url=${ZAP_TARGET} \
                    -d recurse=true \
                    -d inContext=false
                sleep 60 // Attente pour laisser le temps au scan de se lancer
                """
            }
        }

        stage('Générer le rapport') {
            steps {
                echo "Génération du rapport JSON..."
                sh """
                curl -X GET "http://localhost:${ZAP_PORT}/OTHER/core/other/jsonreport/" -o zap_report.json
                """
            }
        }

        stage('Arrêter ZAP') {
            steps {
                script {
                    echo "Arrêt de ZAP..."
                    // Arrêt du conteneur ZAP
                    sh 'docker stop ${ZAP_CONTAINER_NAME}'
                    // Suppression du conteneur ZAP pour libérer les ressources
                    sh 'docker rm ${ZAP_CONTAINER_NAME}'
                }
            }
        }

        stage('Archiver les résultats') {
            steps {
                echo "Archivage du rapport..."
                // Archivage du rapport JSON généré dans Jenkins pour consultation
                archiveArtifacts allowEmptyArchive: true, artifacts: 'zap_report.json', onlyIfSuccessful: true
            }
        }
    }

    post {
        always {
            // Nettoyage supplémentaire si nécessaire, par exemple supprimer les volumes Docker inutiles
            echo "Nettoyage des ressources Docker..."
            sh 'docker system prune -f'
        }
    }
}
