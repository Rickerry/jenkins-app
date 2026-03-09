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
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }

        stage('SAST Scan') {
            steps {
                script {
                    def scannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=jenkins-app \
                                -Dsonar.sources=. \
                                -Dsonar.exclusions=venv/**,reports/**,**/__pycache__/** \
                                -Dsonar.host.url=http://192.168.56.1:9000 \
                                -Dsonar.token=squ_994c71699ff4649a5964572e0656981f9a4969a7
                        """
                    }
                }
            }
        }

       

        stage('SCA Scan') {
            steps {
                sh '''
                    rm -rf reports
                    mkdir -p reports
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
                    publishHTML([
                        reportDir: 'reports',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'OWASP Dependency-Check Report',
                        keepAll: true,
                        alwaysLinkToLastBuild: true,
                        allowMissing: true
                    ])
                    archiveArtifacts artifacts: 'reports/dependency-check-report.html', fingerprint: true, allowEmptyArchive: true
                }
            }
        }
    }

    post {
        success { echo '✅ Pipeline exécuté avec succès !' }
        failure { echo '❌ Le pipeline a échoué !' }
        always { cleanWs() }
    }
}
