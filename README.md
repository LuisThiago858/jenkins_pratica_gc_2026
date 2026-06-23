# Jenkins Prática GC 2026

Este projeto foi desenvolvido como parte da **Segunda Atividade Prática (2026-1) - Jenkins**, com o objetivo de montar uma versão inicial de um processo de **build e testes automáticos** utilizando o **Jenkins integrado com o GitHub**.

## Objetivo

Configurar um pipeline de integração contínua no Jenkins capaz de:

- buscar o código-fonte diretamente do GitHub;
- preparar o ambiente de execução;
- realizar uma verificação de build/compilação;
- executar testes automatizados;
- gerar relatório de cobertura de código;
- demonstrar diferentes cenários de sucesso, falha, instabilidade e execução agendada.

## Tecnologias utilizadas

- Jenkins
- GitHub
- Python
- Pytest
- Pytest-cov
- Git

## Estrutura do projeto

```text
jenkins_pratica_gc_2026/
├── src/
│   ├── __init__.py
│   └── conversor.py
├── tests/
│   └── test_conversor.py
├── Jenkinsfile
└── requirements.txt
```

## Métodos implementados

O projeto possui dois métodos simples de conversão de temperatura.

Arquivo: `src/conversor.py`

```python
def fahrenheit_para_celsius(fahrenheit):
    return (fahrenheit - 32) * 5 / 9


def celsius_para_fahrenheit(celsius):
    return (celsius * 9 / 5) + 32
```

### Método 1

```python
fahrenheit_para_celsius(fahrenheit)
```

Converte uma temperatura de Fahrenheit para Celsius.

Exemplo:

```python
fahrenheit_para_celsius(32)
```

Resultado esperado:

```text
0
```

### Método 2

```python
celsius_para_fahrenheit(celsius)
```

Converte uma temperatura de Celsius para Fahrenheit.

Exemplo:

```python
celsius_para_fahrenheit(100)
```

Resultado esperado:

```text
212
```

## Casos de teste

Os testes foram implementados com `pytest`.

Arquivo: `tests/test_conversor.py`

```python
from src.conversor import fahrenheit_para_celsius, celsius_para_fahrenheit


def test_fahrenheit_para_celsius():
    assert fahrenheit_para_celsius(32) == 0
    assert fahrenheit_para_celsius(212) == 100


def test_celsius_para_fahrenheit():
    assert celsius_para_fahrenheit(0) == 32
    assert celsius_para_fahrenheit(100) == 212
```

Os testes validam os seguintes cenários:

| Função | Entrada | Resultado esperado |
|---|---:|---:|
| `fahrenheit_para_celsius` | `32` | `0` |
| `fahrenheit_para_celsius` | `212` | `100` |
| `celsius_para_fahrenheit` | `0` | `32` |
| `celsius_para_fahrenheit` | `100` | `212` |

## Dependências

Arquivo: `requirements.txt`

```text
pytest
pytest-cov
```

## Como executar localmente

### 1. Clonar o repositório

```bash
git clone https://github.com/LuisThiago858/jenkins_pratica_gc_2026.git
cd jenkins_pratica_gc_2026
```

### 2. Criar ambiente virtual

No Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

No Linux/macOS:

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Instalar dependências

```bash
pip install -r requirements.txt
```

### 4. Executar testes

```bash
pytest -v
```

### 5. Executar testes com cobertura

```bash
pytest --cov=src --cov-report=xml --cov-report=html
```

Após a execução, são gerados:

```text
coverage.xml
htmlcov/
```

## Pipeline Jenkins

O pipeline foi configurado no arquivo `Jenkinsfile`.

```groovy
pipeline {
    agent any

    triggers {
        cron('H/1 * * * *')
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
                bat '''
                    python -m venv venv
                    call venv\\Scripts\\activate.bat
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Verificando compilação/sintaxe...'
                bat '''
                    call venv\\Scripts\\activate.bat
                    set PYTHONPATH=%CD%
                    python -m py_compile src\\conversor.py
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Executando testes...'
                bat '''
                    call venv\\Scripts\\activate.bat
                    set PYTHONPATH=%CD%
                    python -m pytest -v --junitxml=test-results.xml
                '''
            }
        }

        stage('Coverage') {
            steps {
                echo 'Executando cobertura de código...'
                bat '''
                    call venv\\Scripts\\activate.bat
                    set PYTHONPATH=%CD%
                    python -m pytest --cov=src --cov-report=xml --cov-report=html
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
```

## Etapas do pipeline

### Checkout

Busca o código-fonte diretamente do repositório no GitHub.

```groovy
checkout scm
```

### Setup

Cria o ambiente virtual Python e instala as dependências do projeto.

### Build

Executa a verificação de sintaxe do arquivo principal usando:

```bash
python -m py_compile src\conversor.py
```

Essa etapa simula o processo de compilação/build do projeto.

### Test

Executa os testes automatizados com `pytest` e gera o arquivo `test-results.xml`.

### Coverage

Executa os testes com cobertura de código usando `pytest-cov`, gerando os relatórios `coverage.xml` e `htmlcov/`.

## Configuração do Jenkins com GitHub

O Jenkins foi configurado como um projeto do tipo **Pipeline**, utilizando a opção:

```text
Pipeline script from SCM
```

Configuração utilizada:

```text
SCM: Git
Repository URL: https://github.com/LuisThiago858/jenkins_pratica_gc_2026.git
Credentials: None
Branch Specifier: */main
Script Path: Jenkinsfile
```

Com isso, o Jenkins busca automaticamente o `Jenkinsfile` no repositório GitHub e executa as etapas definidas no pipeline.

## Cenários demonstrados

### Cenário 1 - Build manual com sucesso

Neste cenário, o job foi executado manualmente no Jenkins. O build foi concluído com sucesso e todos os testes foram executados corretamente.

Resultado esperado:

```text
Finished: SUCCESS
```

### Cenário 2 - Falha durante a compilação

Neste cenário, foi inserido um erro de sintaxe proposital no código-fonte. A etapa de build identificou o erro durante a verificação com `py_compile`.

Resultado esperado:

```text
Finished: FAILURE
```

### Cenário 3 - Build com sucesso, mas teste falhando

Neste cenário, o código compilou corretamente, mas um dos métodos foi alterado para retornar um valor incorreto. Com isso, os testes automatizados falharam.

O objetivo foi demonstrar um caso em que o problema não está na sintaxe, mas sim no comportamento da aplicação.

Resultado esperado:

```text
UNSTABLE
```

### Cenário 4 - Build agendado

Neste cenário, o Jenkins foi configurado para executar o job automaticamente por meio do agendamento definido no `Jenkinsfile`.

Exemplo de configuração:

```groovy
triggers {
    cron('H/1 * * * *')
}
```

A execução ocorre sem intervenção manual do usuário.

Resultado esperado:

```text
Finished: SUCCESS
```

### Cenário bônus - Cobertura de código

Neste cenário, foi executada a métrica de cobertura de código utilizando `pytest-cov`.

Comando utilizado no pipeline:

```bash
python -m pytest --cov=src --cov-report=xml --cov-report=html
```

Arquivos gerados:

```text
coverage.xml
htmlcov/
```

## Links dos vídeos

| Cenário | Descrição | Link |
|---|---|---|
| Cenário 1 | Build manual com sucesso e testes passando | Inserir link do vídeo |
| Cenário 2 | Build falhando durante a compilação | Inserir link do vídeo |
| Cenário 3 | Build com sucesso, mas testes falhando/instável | Inserir link do vídeo |
| Cenário 4 | Build agendado executando automaticamente | Inserir link do vídeo |
| Bônus | Build com cobertura de código | Inserir link do vídeo |

## Link do projeto no GitHub

```text
https://github.com/LuisThiago858/jenkins_pratica_gc_2026
```

## Conclusão

A atividade demonstrou a configuração inicial de um processo de integração contínua utilizando Jenkins e GitHub. O pipeline criado permite automatizar etapas importantes do desenvolvimento, como checkout do código, preparação do ambiente, verificação de build, execução de testes e geração de cobertura.

Com os cenários executados, foi possível observar diferentes comportamentos do Jenkins diante de código correto, erro de compilação, falha de teste, execução agendada e uso de métrica de cobertura de código.
