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
                sh 'echo "SAST Scan désactivé - installation de SonarQube requise"'
            }
        }

        stage('SCA Scan') {
            steps {
                sh '''
                    # Nettoyer l'ancien dossier reports
                    rm -rf reports
                    mkdir -p reports
                    
                    # Forcer la sortie détaillée pour voir l'erreur
                    /usr/local/bin/dependency-check.sh \\
                        --project "TP-Jenkins" \\
                        --scan . \\
                        --format HTML \\
                        --failOnCVSS 7 \\
                        --out reports \\
                        --data /opt/dependency-check-final/data \\
                        --purge 2>&1 | tee dependency-check.log
                    
                    # Afficher le log pour debug
                    cat dependency-check.log
                '''
            }
            post {
                always {
                    // Publier le rapport HTML s'il existe
                    publishHTML([
                        reportDir: 'reports',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'OWASP Dependency-Check Report',
                        keepAll: true,
                        alwaysLinkToLastBuild: true,
                        allowMissing: true
                    ])
                    
                    // Archiver le rapport s'il existe
                    archiveArtifacts artifacts: 'reports/dependency-check-report.html', fingerprint: true, allowEmptyArchive: true
                    
                    // Archiver le log de debug
                    archiveArtifacts artifacts: 'dependency-check.log', fingerprint: true, allowEmptyArchive: true
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
