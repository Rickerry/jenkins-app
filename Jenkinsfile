pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Rickerry/jenkins-app.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest
                '''
            }
        }

        stage('SAST Scan') {
            steps {
                // Désactivé temporairement - à configurer avec SonarQube plus tard
                sh 'echo "SAST Scan désactivé - installation de SonarQube requise"'
            }
        }

        stage('SCA Scan') {
            steps {
                dependencyCheck odcInstallation: 'DP-Check',
                    additionalArguments: '''
                        --project "TP-Jenkins"
                        --scan .
                        --format HTML
                        --failOnCVSS 7
                        --out reports
                    '''
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'reports/dependency-check-report.xml'
                    publishHTML([
                        reportDir: 'reports',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'OWASP Dependency-Check Report',
                        keepAll: true,
                        alwaysLinkToLastBuild: true,
                        allowMissing: false
                    ])
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Le pipeline a échoué !'
        }
        success {
            echo '✅ Pipeline exécuté avec succès !'
        }
    }
}
