pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub_vinhbh'
        IMAGE_NAME = 'vinhbh/dramatiq_student'
        DRAMATIQ_DOCKERFILE_PATH = '.'
    }
    stages {
        stage('Clone') {
            steps {
                script {
                    echo "Clone code from branch ${env.BRANCH_NAME}"
                    git branch: env.BRANCH_NAME, url: 'https://github.com/Vinh1507/dramatiq-background-job'
                }
                script {
                    def tagVersion = sh(script: 'git describe --tags --abbrev=0', returnStdout: true).trim()
                    env.TAG_NAME = tagVersion
                    echo "Tag version: ${env.TAG_NAME}"
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    // docker.build('vinhbh/simple_image_jenkins:lastest', '.')
                    echo "Image version: ${env.IMAGE_NAME}:${env.TAG_NAME}"
                    sh "docker build --no-cache -t ${env.IMAGE_NAME}:${env.TAG_NAME} ${DRAMATIQ_DOCKERFILE_PATH}"
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                // Login to Docker Hub
                withCredentials([usernamePassword(credentialsId: 'dockerhub_vinhbh', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                }
                // Push Docker image to Docker Hub
                sh "docker push ${env.IMAGE_NAME}:${env.TAG_NAME}"
            }
        }
        stage('Deploy dramatiq application') {
            steps {
                ansiblePlaybook credentialsId: 'ansible-private-key', 
                disableHostKeyChecking: true, 
                installation: 'ansible2', 
                inventory: './ansible/inventory.yml', 
                playbook: './ansible/playbooks/dramatiq.yml', 
                vaultTmpPath: '',
                extras: "-e DRAMATIQ_IMAGE_VERSION=${env.TAG_NAME}"
            }
        }
    }
}
