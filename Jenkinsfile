pipeline {
    agent any
    environment {
        ADMIN_USER = 'OnsDj'
        ADMIN_PASSWORD = 'OnsDj'
    }
    stages {
        stage('Checkout') {
            steps {
                withCredentials([string(credentialsId: '55558', variable: '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')]) {
                    // Supprimer le répertoire existant avant de cloner
                    sh 'rm -rf Jenkins'  // Supprime le répertoire 'Jenkins' s'il existe
                    echo 'Cloning repository...'
                    // Cloner le dépôt avec l'authentification via token
                    sh 'git clone https://x-access-token:${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}@github.com/OnsDJELASSI/Jenkins.git'
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Building...'
                // Ajoutez vos étapes de construction ici (par exemple, mvn clean install)
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Ajoutez vos étapes de test ici
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Créez l'image Docker et déployez l'application
                sh 'docker build -t votre-image .'
                sh 'docker run -d -p 8080:8080 votre-image'
            }
        }
        stage('Security Test with OWASP ZAP') {
            steps {
                script {
                    // Lancer OWASP ZAP via Docker pour scanner l'application déployée
                    echo 'Starting OWASP ZAP security test...'

                    // Démarrer le container OWASP ZAP
                    sh 'docker run -d --name zaproxy -u zaproxy owasp/zap2docker-stable'
                    
                    // Attendre que ZAP soit prêt
                    sh 'sleep 10'

                    // Lancer un scan de sécurité sur l'URL de votre application
                    sh 'docker exec zaproxy zap-cli quick-scan --start-url http://localhost:8080'

                    // Exporter le rapport en format HTML
                    sh 'docker exec zaproxy zap-cli report -o /zap/wrk/zap_report.html -f html'

                    // Copier le rapport dans le workspace Jenkins
                    sh 'docker cp zaproxy:/zap/wrk/zap_report.html ./zap_report.html'

                    // Arrêter OWASP ZAP après le scan
                    sh 'docker stop zaproxy'
                    sh 'docker rm zaproxy'

                    // Interpréter les résultats du scan (par exemple, échouer le build si des vulnérabilités critiques sont trouvées)
                    script {
                        def report = readFile('zap_report.html')
                        // Exemple : vérifier si le rapport contient un avertissement de vulnérabilité
                        if (report.contains("High")) {
                            error "Security issues found in the application, build aborted."
                        } else {
                            echo "No critical security issues found."
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build and security scan completed successfully.'
        }
        failure {
            echo 'Build or security scan failed.'
        }
    }
}
