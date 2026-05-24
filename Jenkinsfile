pipeline {
    agent any
    // triggers {
    //     cron('H/2 * * * *')
    // }

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
                bat '''
                    python -m venv venv
                    call venv\\Scripts\\activate.bat
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Build') {
            steps {
                echo 'Verificando compilação/sintaxe...'
                bat '''
                    call venv\\Scripts\\activate.bat
                    python -m py_compile src\\conversor.py
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Executando testes...'
                bat '''
                    call venv\\Scripts\\activate.bat
                    pytest -v --junitxml=test-results.xml
                '''
            }
        }
        stage('Coverage') {
            steps {
                echo 'Executando cobertura de código...'
                bat '''
                    call venv\\Scripts\\activate.bat
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