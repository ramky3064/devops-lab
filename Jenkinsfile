pipeline {
    agent any
    environment {
        IMAGE_NAME = 'taskapi'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        CONTAINER  = 'taskapi'
    }

    agent any
    environment {
        PATH = "C:\\Users\\krish\\AppData\\Local\\Python\\pythoncore-3.15-64;C:\\Users\\krish\\AppData\\Local\\Python\\pythoncore-3.15-64\\Scripts;${env.PATH}"
    }
    stages {
        ...
    }

    options {
        timestamps()
        timeout(time: 20, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    python -m venv venv
                    venv\\Scripts\\python -m pip install --upgrade pip
                    venv\\Scripts\\pip install -r requirements-dev.txt
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                bat '''
                    if not exist reports mkdir reports
                    venv\\Scripts\\pytest tests --junitxml=reports/junit.xml --cov=app --cov-report=xml:reports/coverage.xml --cov-report=term
                '''
            }
            post {
                always { junit 'reports/junit.xml' }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script { scannerHome = tool 'SonarScanner' }
                withSonarQubeEnv('sonarqube') {
                    bat "\"${scannerHome}\\bin\\sonar-scanner.bat\""
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

        stage('Docker Build') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%IMAGE_TAG% -t %IMAGE_NAME%:latest .'
            }
        }

        stage('Deploy (local container)') {
            steps {
                bat '''
                    docker rm -f %CONTAINER% 2>nul
                    docker run -d --name %CONTAINER% -p 5000:5000 %IMAGE_NAME%:%IMAGE_TAG%
                    ping -n 8 127.0.0.1 >nul
                    curl -fs http://localhost:5000/health
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
        }
        success { echo 'Pipeline succeeded' }
        failure { echo 'Pipeline failed' }
    }
}
