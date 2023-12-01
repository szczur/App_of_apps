def frontendImage="szczur123/panda-frontend"
def backendImage="szczur123/panda-backend"
def dockerRegistry=""
def registryCredentials="dockerhub"

pipeline {
    agent {
        label 'agent'
    }
    stages {
        stage('Get Code') {
            steps {
                //echo 'Hello World'
                checkout scm
            }
        }
    }
}
