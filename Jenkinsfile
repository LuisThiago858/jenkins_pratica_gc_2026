pipeline {
    agent any
    triggers {

        cron('H/2 * * * *')
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Baixando código do GitHub...'
                checkout scm
            }
        }
        stage('Setup') {
            steps {
                echo 'Preparando ambiente Python...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Build') {
            steps {
                echo 'Verificando compilação/sintaxe...'
                sh '''
                    . venv/bin/activate
                    python3 -m py_compile src/conversor.py
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Executando testes...'
                sh '''
                    . venv/bin/activate
                    pytest -v --junitxml=test-results.xml
                '''
            }
        }
        stage('Coverage') {
            steps {
                echo 'Executando cobertura de código...'
                sh '''
                    . venv/bin/activate
                    pytest --cov=src --cov-report=xml --cov-report=html
                '''
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'test-results.xml'
            archiveArtifacts artifacts: 'htmlcov/**, coverage.xml', allowEmptyArchive: true
        }
    }
}