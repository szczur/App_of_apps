def frontendImage="szczur123/panda-frontend"
def backendImage="szczur123/panda-backend"
def backendDockerTag=""
def frontendDockerTag=""
def dockerRegistry=""
def registryCredentials="dockerhub"

pipeline {
    agent {
        label 'agent'
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES=1
    }
    parameters {
        string(defaultValue: 'default', name: 'backendDockerTag')
        string(defaultValue: 'default', name: 'frontendDockerTag')
    }
    stages {
        stage('Get code') {
            steps {
                //echo 'Hello World'
                checkout scm
            }
        }
        stage('Adjust version') {
            steps {
                script{
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }
        stage('Remove containers') {
            steps {
                sh "docker rm -f frontend"
                sh "docker rm -f backend"
            }
        }
        stage('Deploy application') {
            steps {
                script {
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                             "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                       docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                            sh "docker-compose up -d"
                        }
                    }
                }
            }
        }
        stage('Selenium test') {
            steps {
                sh "pip3 install -r test/selenium/requirements.txt"
                sh "python -m pytest test/selenium/frontendTest.py"
            }
        }
    }
    post {
        always {
            sh "docker-compose down"
            cleanWs()
        }
    }
}
