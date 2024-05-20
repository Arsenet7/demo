pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Enter the GitHub branch name')
        string(name: 'DOCKER_PORT', defaultValue: '9000', description: 'Enter the port for Docker container')
    }

    environment {
        DOCKER_IMAGE = 'devopseasylearning/thursday:v5'
    }

    stages {
        stage('Install Docker') {
            steps {
                script {
                    // Install Docker
                    sh 'curl -fsSL https://get.docker.com -o get-docker.sh'
                    sh 'sudo sh get-docker.sh'
                }
            }
        }
        
        stage('Add Jenkins to Docker Group') {
            steps {
                script {
                    def jenkinsUser = sh(script: 'whoami', returnStdout: true).trim()
                    sh "sudo usermod -aG docker ${jenkinsUser}"
                    sh 'sudo systemctl restart docker.service'
                }
            }
        }

        stage('Clone') {
            steps {
                git url: 'https://github.com/Royce237/s5royce.git', branch: "${params.GIT_BRANCH}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Ensure Jenkins has permission to interact with Docker
                    sh 'sudo chmod 666 /var/run/docker.sock'
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    sh "docker images"
                }
            }
        }

        stage('Log in to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-id', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh "docker push ${DOCKER_IMAGE}"
                sh "docker run -d -p ${params.DOCKER_PORT}:80 ${DOCKER_IMAGE}"
                sh 'docker ps'
                sh 'docker images'
                sh 'docker ps -a'
            }
        }
    }
}
