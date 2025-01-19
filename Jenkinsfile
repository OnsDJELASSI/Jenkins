pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Commandes pour construire votre application
                echo 'Building...'
                // Exemple : Compilation du projet
                sh 'make' // ou une autre commande de construction
            }
        }
        stage('Test') {
            steps {
                // Commandes pour tester votre application
                echo 'Testing...'
                // Exemple : Tests unitaires
                sh 'make test' // ou une autre commande de test
            }
        }
        stage('Deploy') {
            steps {
                // Commandes pour déployer votre application
                echo 'Deploying...'
                // Exemple : Déploiement de l'application
                sh 'make deploy' // ou une autre commande de déploiement
            }
        }
        stage('Security Test with OWASP ZAP') {
            steps {
                script {
                    echo 'Starting OWASP ZAP security test...'

                    // Démarrer le conteneur OWASP ZAP
                    sh 'docker run -d --name zaproxy securecodebox/zap'

                    // Attendre que OWASP ZAP soit prêt
                    sh 'sleep 10'

                    // Lancer un scan rapide sur l'URL de l'application
                    sh 'docker exec zaproxy zap-cli quick-scan --start-url http://localhost:8080'

                    // Exporter le rapport en HTML
                    sh 'docker exec zaproxy zap-cli report -o /zap/wrk/zap_report.html -f html'

                    // Copier le rapport dans le répertoire Jenkins
                    sh 'docker cp zaproxy:/zap/wrk/zap_report.html ./zap_report.html'

                    // Arrêter et supprimer le conteneur OWASP ZAP
                    sh 'docker stop zaproxy'
                    sh 'docker rm zaproxy'

                    echo 'Security test with OWASP ZAP completed.'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
            // Archiver le rapport généré dans Jenkins
            archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
        }
    }
}
