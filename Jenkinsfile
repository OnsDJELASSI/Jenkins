pipeline {
    agent any

    environment {
        ZAP_DAEMON_URL = "http://localhost:7075"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Prepare ZAP') {
            steps {
                script {
                    echo "Vérification si ZAP est déjà lancé..."
                    sh '''
                    if ! docker ps -q --filter "name=zap"; then
                        echo "Démarrage de ZAP en mode daemon sur le port 7075..."
                        docker run -u zap -d -p 7075:7075 --name zap zaproxy/zap-stable:2.14.0 zap.sh -daemon -port 7075

                        echo "Attente que ZAP soit prêt..."
                        for i in {1..10}; do
                          if curl -s http://localhost:7075 > /dev/null; then
                              echo "ZAP est prêt."
                              break
                          fi
                          echo "ZAP pas encore prêt, nouvelle tentative dans 5s..."
                          sleep 5
                        done
                    else
                        echo "ZAP est déjà en cours d'exécution."
                    fi
                    '''
                }
            }
        }

        stage('Run ZAP Scan') {
            steps {
                script {
                    echo "Lancement du scan OWASP ZAP..."
                    sh '''
                    curl -X GET ${ZAP_DAEMON_URL}/JSON/ascan/action/scan \
                        -d "url=http://example.com" \
                        -d "recurse=true" \
                        -d "inContext=false"
                    '''
                }
            }
        }

        stage('Wait for Scan Completion') {
            steps {
                script {
                    echo "Attente de la fin du scan..."
                    sleep(time: 60, unit: 'SECONDS')  // ajustable
                }
            }
        }

        stage('Archive Results') {
            steps {
                script {
                    echo "Archivage des résultats du scan..."
                    sh '''
                    echo "Copie des fichiers JSON depuis le conteneur..."
                    docker cp zap:/zap/wrk/. . || echo "Aucun fichier JSON trouvé"
                    '''
                    archiveArtifacts artifacts: '**/*.json', allowEmptyArchive: true
                }
            }
        }

        stage('Stop ZAP') {
            steps {
                script {
                    echo "Arrêt de ZAP..."
                    sh '''
                    docker stop zap || echo "Erreur à l'arrêt de ZAP"
                    docker rm zap || echo "Erreur à la suppression du conteneur ZAP"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Nettoyage de l'environnement..."
            sh 'docker ps -a -q --filter "name=zap" | xargs -r docker rm -f || true'
        }
    }
}
