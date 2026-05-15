pipeline {
    agent any

    tools {
        maven 'Default Maven'
    }

    environment {

        REMOTE_HOST = '172.31.47.158'
        REMOTE_USER = 'ec2-user'

        REMOTE_APP_DIR = '/home/ec2-user/poppy-app'

        IMAGE_NAME = 'poppy-app'
        CONTAINER_NAME = 'poppy-container'
    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                url: 'https://github.com/Gopireddymadhava/sample-project-test.git'
            }
        }

        stage('Build and SonarQube Analysis') {

            steps {

                sh 'chmod +x mvnw || true'

                withSonarQubeEnv('Sonar-Server') {


                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=Giva-Proj-Scanner
                    '''
                }
            }
        }

        stage('Copy files to Docker EC2') {

            steps {

                sshagent(['docker-ec2-key']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ${REMOTE_USER}@${REMOTE_HOST} \
                        "mkdir -p ${REMOTE_APP_DIR}"

                        scp -o StrictHostKeyChecking=no -r \
                        * \
                        ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_APP_DIR}/
                    '''
                }
            }
        }

        stage('Build Docker Image on Docker EC2') {

            steps {

                sshagent(['docker-ec2-key']) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ${REMOTE_USER}@${REMOTE_HOST} "

                            cd ${REMOTE_APP_DIR}

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
                        ssh -o StrictHostKeyChecking=no \
                        ${REMOTE_USER}@${REMOTE_HOST} "

                            docker stop ${CONTAINER_NAME} || true

                            docker rm ${CONTAINER_NAME} || true

                            docker run -d \
                            --restart always \
                            --name ${CONTAINER_NAME} \
                            -p 8085:8085 \
                            ${IMAGE_NAME}:latest

                        "
                    '''
                }
            }
        }
    }

    post {

        success {
            echo 'Application deployed successfully 🚀'
        }

        failure {
            echo 'Pipeline failed ❌'
        }
    }
}



