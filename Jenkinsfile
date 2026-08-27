  GNU nano 6.2                                                                               Jenkinsfile                                                                                         
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

        stage('Install & Test') {
            steps {
                sh 'python3 -m pip install --user Flask pytest'
                sh 'python3 -m pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'pkill -f "python3 app.py" || true'
                sh 'nohup python3 app.py > app.log 2>&1 &'
            }
        }
    }
}




