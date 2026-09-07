pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git credentialsId: 'github-token',
                    url: 'https://github.com/Adnan-Khairullah/hello-python.git',
                    branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'python3 -m pip install --user -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to App VM') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'gce-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i "$SSH_KEY" \
                            -o StrictHostKeyChecking=no \
                            "$SSH_USER@10.160.0.9" \
                            "mkdir -p ~/hello-python"

                        scp -i "$SSH_KEY" \
                            -o StrictHostKeyChecking=no \
                            app.py requirements.txt \
                            "$SSH_USER@10.160.0.9:~/hello-python/"

                        ssh -i "$SSH_KEY" \
                            -o StrictHostKeyChecking=no \
                            "$SSH_USER@10.160.0.9" \
                            "cd ~/hello-python && \
                             python3 -m pip install --user -r requirements.txt && \
                             pkill -f 'python3 app.py' || true; \
                             nohup python3 app.py > app.log 2>&1 &"
                    '''
                }
            }
        }
    }
}
