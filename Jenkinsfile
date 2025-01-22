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
                withCredentials([string(credentialsId: '55558', variable: 'GIT_TOKEN')]) {
                    echo 'Cloning repository...'
                    sh 'rm -rf Jenkins'  // Supprimer le dossier existant
                    sh 'git clone https://x-access-token:${GIT_TOKEN}@github.com/OnsDJELASSI/Jenkins.git'
                    sh 'ls -la Jenkins'  // Vérifier si pom.xml est bien cloné
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building project...'
                sh '''
                    cd Jenkins
                    mvn clean install -U
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh '''
                    cd Jenkins
                    mvn test
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
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
                    echo 'Starting OWASP ZAP security test...'

                    // Démarrer OWASP ZAP
                    sh 'docker run -d --name zaproxy owasp/zap2docker-stable'
                    sh 'sleep 10'  // Attendre que OWASP ZAP démarre

                    // Lancer un scan rapide sur l'application
                    sh '''
                        docker exec zaproxy zap-cli quick-scan --start-url http://localhost:8080
                        docker exec zaproxy zap-cli report -o /zap/wrk/zap_report.html -f html
                        docker cp zaproxy:/zap/wrk/zap_report.html ./zap_report.html
                    '''

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
            archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
        }
    }
}
