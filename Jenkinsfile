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
    tools {
        terraform 'Terraform'
    }
    parameters {
        string(defaultValue: '', name: 'backendDockerTag')
        string(defaultValue: '', name: 'frontendDockerTag')
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
                sh "python3 -m pytest test/selenium/frontendTest.py"
            }
        }
        stage('Run terraform') {
            steps {
                dir('Terraform') {
                    git branch: 'main', url: 'https://github.com/szczur/Terraform.git'
                    withAWS(credentials:'AWS', region: 'us-east-1') {
                        sh 'terraform init -backend-config=bucket=pawel-szczurek-panda-devops-core-15'
                        sh 'terraform apply -auto-approve -var bucket_name=pawel-szczurek-panda-devops-core-15'
                    } 
                }
            }
        }
        stage('Run Ansible') {
            steps {
                script {
                    sh "pip3 install -r requirements.txt"
                    sh "ansible-galaxy install -r requirements.yml"
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                            "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                        ansiblePlaybook inventory: 'inventory', playbook: 'playbook.yml'
                        }
                } 
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
