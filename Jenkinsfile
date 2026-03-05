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
                sh 'echo "sonar-scanner (à configurer)"'
            }
        }

        stage('SCA Scan') {
            steps {
                sh '''
                dependency-check.sh \\
                  --project "TP-Jenkins" \\
                  --scan . \\
                  --format HTML \\
                  --failOnCVSS 7 \\
                  --out reports \\
                  --purge
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/dependency-check-report.html', fingerprint: true
                }
            }
        }
    }

    post {
        failure {
            echo 'Le pipeline a échoué !'
        }
        success {
            echo 'Pipeline exécuté avec succès !'
        }
    }
}
