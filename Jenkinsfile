pipeline {
    agent any

    environment {
        ZAP_DAEMON_URL = "http://localhost:7075" // L'URL du serveur ZAP en mode daemon
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Cloner le projet depuis le dépôt Git
                    checkout scm
                }
            }
        }

        stage('Prepare ZAP') {
            steps {
                script {
                    // Vérifier si ZAP est déjà en fonctionnement
                    echo "Vérification si ZAP est déjà lancé..."
                    sh '''
                    if ! docker ps -q --filter "name=zap"; then
                        echo "Démarrage de ZAP en mode daemon..."
                        docker run -u zap -p 7075:8080 --name zap zaproxy/zap-stable:2.14.0 zap.sh -daemon
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
                    // Lancer un scan avec ZAP sur un site de test (exemple: http://example.com)
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
                    // Attendre que le scan soit terminé
                    echo "Attente de la fin du scan..."
                    sleep(time: 60, unit: 'SECONDS')  // Attendre 60 secondes (à ajuster en fonction de la durée du scan)
                }
            }
        }

        stage('Stop ZAP') {
            steps {
                script {
                    // Arrêter le conteneur ZAP après le scan
                    echo "Arrêt de ZAP..."
                    sh 'docker stop zap && docker rm zap'
                }
            }
        }

        stage('Archive Results') {
            steps {
                script {
                    // Archiver les résultats du scan (ajoute la logique pour récupérer les résultats de ZAP)
                    echo "Archivage des résultats du scan..."
                    sh 'docker cp zap:/zap/wrk/*.json .'
                    archiveArtifacts artifacts: '**/*.json', allowEmptyArchive: true
                }
            }
        }
    }

    post {
        always {
            // Nettoyage (optionnel)
            echo "Nettoyage de l'environnement..."
            sh 'docker ps -a -q --filter "name=zap" | xargs -r docker rm -f'
        }
    }
}
