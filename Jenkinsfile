pipeline {
    agent any

    stages {
        stage('Préparation') {
            steps {
                echo 'Nettoyage du workspace...'
                cleanWs()
            }
        }

        stage('Cloner le dépôt') {
            steps {
                git 'https://github.com/OnsDJELASSI/Jenkins.git'
            }
        }

        stage('Lancer OWASP ZAP') {
            steps {
                script {
                    echo 'Vérification si ZAP est déjà en cours d\'exécution...'
                    def zapContainerRunning = sh(script: "docker ps -q -f name=zap", returnStdout: true).trim()
                    if (zapContainerRunning) {
                        echo 'ZAP est déjà en cours d\'exécution.'
                    } else {
                        echo 'Démarrage de ZAP...'
                        sh '''
                        docker run -u zap -d -p 7075:7075 --name zap \
                          zaproxy/zap-stable:2.14.0 \
                          zap.sh -daemon -host 0.0.0.0 -port 7075 \
                          -config api.addrs.addr.name='.*' \
                          -config api.addrs.addr.regex=true \
                          -config api.disablekey=true
                        '''
                        echo 'Attente du démarrage de ZAP...'
                        sleep 30
                    }
                }
            }
        }

        stage('Scan avec OWASP ZAP') {
            steps {
                script {
                    echo 'Lancement du scan OWASP ZAP...'
                    sh '''
                    curl "http://localhost:7075/JSON/ascan/action/scan/?url=https://testphp.vulnweb.com&recurse=true&inContext=false"
                    sleep 60
                    '''
                }
            }
        }

        stage('Arrêter ZAP & Récupérer les résultats') {
            steps {
                script {
                    echo 'Arrêt de ZAP...'
                    sh 'docker stop zap || true'

                    echo 'Archivage des résultats du scan...'
                    sh 'docker cp zap:/zap/wrk . || true'
                }
            }
        }
    }

    post {
        always {
            echo 'Nettoyage final...'
            sh 'docker rm zap || true'
            archiveArtifacts artifacts: 'wrk/*.json', allowEmptyArchive: true
        }
    }
}
