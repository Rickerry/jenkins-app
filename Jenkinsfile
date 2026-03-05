pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/user/project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest'
            }
        }

        stage('SAST Scan') {
            steps {
                sh 'sonar-scanner'
            }
        }

        stage('SCA Scan') {
            steps {
                sh '''
                dependency-check.sh \
                  --project "TP-Jenkins" \
                  --scan . \
                  --format HTML \
                  --failOnCVSS 7 \
                  --out reports
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
    }
}
