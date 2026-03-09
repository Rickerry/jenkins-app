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
        withSonarQubeEnv('SonarQube') {
            sh '''
                . venv/bin/activate
                sonar-scanner \
                -Dsonar.projectKey=jenkins-app \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://localhost:9000 \
                -Dsonar.login=$SONAR_AUTH_TOKEN
            '''
        }
    }
}

        stage('SCA Scan') {
            steps {
                sh '''
                    # Nettoyer l'ancien dossier reports
                    rm -rf reports
                    mkdir -p reports
                    
                    # Lancer OWASP Dependency-Check SANS purge
                    /usr/local/bin/dependency-check.sh \\
                        --project "TP-Jenkins" \\
                        --scan . \\
                        --format HTML \\
                        --failOnCVSS 7 \\
                        --out reports \\
                        --data /opt/dependency-check-final/data
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
