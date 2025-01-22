pipeline {
    agent any

    environment {
        ADMIN_USER = 'OnsDj'
        ADMIN_PASSWORD = 'OnsDj'
    }

    tools {
        maven 'Maven 3'
    }

    stages {
        stage('Checkout') {
            steps {
                withCredentials([string(credentialsId: '55558', variable: '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')]) {
                    echo '🚀 Clonage du repository...'
                    sh 'rm -rf Jenkins'  // Supprimer le dossier existant
                    sh 'git clone https://x-access-token:${GIT_TOKEN}@github.com/OnsDJELASSI/Jenkins.git'
                    sh 'ls -la Jenkins'  // Vérifier si le pom.xml est bien présent
                }
            }
        }

        stage('Build') {
            steps {
                echo '🔧 Construction du projet...'
                sh '''
                    cd Jenkins
                    mvn clean install -U
                '''
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Exécution des tests...'
                sh '''
                    cd Jenkins
                    mvn test
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Déploiement de l’application...'
                sh '''
                    cd Jenkins
                    docker build -t my-app-image .
                    docker run -d -p 8080:8080 --name my-app-container my-app-image
                '''
            }
        }

        stage('Security Test with OWASP ZAP') {
            steps {
                script {
                    echo '🛡️ Démarrage du test de sécurité avec OWASP ZAP...'

                    // Vérifier et supprimer un ancien conteneur ZAP
                    sh 'docker stop zaproxy || true'
                    sh 'docker rm zaproxy || true'

                    // Lancer OWASP ZAP en mode daemon
                    sh 'docker run -d --name zaproxy -p 8090:8090 owasp/zap2docker-stable zap.sh -daemon -port 8090'

                    // Attendre que ZAP démarre
                    sh 'sleep 15'

                    // Lancer un scan rapide avec OWASP ZAP
                    sh 'docker exec zaproxy zap-cli quick-scan --start-url http://localhost:8080'

                    // Générer un rapport HTML
                    sh 'docker exec zaproxy zap-cli report -o /zap/wrk/zap_report.html -f html'

                    // Copier le rapport dans Jenkins
                    sh 'docker cp zaproxy:/zap/wrk/zap_report.html ./zap_report.html'

                    // Arrêter et supprimer le conteneur ZAP
                    sh 'docker stop zaproxy'
                    sh 'docker rm zaproxy'

                    echo '✅ Test de sécurité terminé avec succès.'
                }
            }
        }
    }

    post {
        always {
            echo '📌 Fin de la pipeline.'
            archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
        }
    }
}
