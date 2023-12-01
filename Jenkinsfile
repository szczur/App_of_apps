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
    }
}
