pipeline {
    agent any

    tools {
        maven 'Default Maven'
    }

    environment {
        REMOTE_HOST = '107.23.64.221'
        REMOTE_USER = 'ec2-user'
        REMOTE_APP_DIR = '/home/ec2-user/poppy-app'
        IMAGE_NAME = 'poppy-app'
        CONTAINER_NAME = 'poppy-container'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Gopireddymadhava/jenkins_sonarqube_project.git'
            }
        }

        stage('Build and SonarQube Analysis') {
            steps {
                dir('poppy') {
                    sh 'chmod +x mvnw'
                    withSonarQubeEnv('Sonar-Server') {
                        sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=Onix-Website-Scan'
                    }
                }
            }
        }

        stage('Copy files to Docker EC2') {
            steps {
                sshagent(['docker-ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} "mkdir -p ${REMOTE_APP_DIR}"
                        scp -o StrictHostKeyChecking=no -r poppy/* ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_APP_DIR}/
                    '''
                }
            }
        }

        stage('Build Docker Image on Docker EC2') {
            steps {
                sshagent(['docker-ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} "
                            cd ${REMOTE_APP_DIR} &&
                            docker build -t ${IMAGE_NAME}:latest .
                        "
                    '''
                }
            }
        }

        stage('Run Container on Docker EC2') {
            steps {
                sshagent(['docker-ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} "
                            docker stop ${CONTAINER_NAME} || true &&
                            docker rm ${CONTAINER_NAME} || true &&
                            docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${IMAGE_NAME}:latest
                        "
                    '''
                }
            }
        }
    }
}
