 pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Commandes pour construire votre application
                echo 'Building...'
                // Par exemple, exécuter une commande de construction
                sh 'make' // ou une autre commande de construction
            }
        }
        stage('Test') {
            steps {
                // Commandes pour tester votre application
                echo 'Testing...'
                // Par exemple, exécuter des tests unitaires
                sh 'make test' // ou une autre commande de test
            }
        }
        stage('Deploy') {
            steps {
                // Commandes pour déployer votre application
                echo 'Deploying...'
                // Par exemple, déployer sur un serveur
                sh 'make deploy' // ou une autre commande de déploiement
            }
        }
    }
}
