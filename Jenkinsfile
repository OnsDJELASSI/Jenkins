pipeline {
    agent any

    environment {
        // Définir des variables d'environnement si nécessaire
        ADMIN_USER = 'OnsDj'
        ADMIN_PASSWORD = 'OnsDj'
    }

    tools {
        maven 'Maven 3'  // Nom de l'outil Maven configuré dans Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                withCredentials([string(credentialsId: '55558', variable: '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')]) {
                    echo 'Cloning repository...'
                    sh 'rm -rf Jenkins'  // Supprimer le répertoire Jenkins existant
                    sh 'git clone https://x-access-token:${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}@github.com/OnsDJELASSI/Jenkins.git'
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean install'  // Exécuter Maven pour construire le projet
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'  // Exécuter les tests unitaires avec Maven
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh 'docker build -t your-image .'  // Créer une image Docker
                sh 'docker run -d -p 8080:8080 your-image'  // Lancer l'application dans un conteneur Docker
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
