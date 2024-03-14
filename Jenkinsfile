pipeline {
        agent {
                node {
                    label "${JENKINS_SLAVE_LABEL}"
                }
            }
    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-cred'
        SLACK_CREDENTIALS = 'slack-bot'
        DOCKER_IMAGE_NAME = 'vhmds/front-end'
        SLACK_CHANNEL = '#devops'
        JENKINS_SLAVE_LABEL = 'docker-remote'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build Docker image
                    def dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}")

                    // Push Docker image to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_HUB_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Notify Slack') {
            steps {
                script {
                    // Notify Slack about the successful build
                    slackSend(color: '#00FF00', message: "Docker image build successful! \n${BUILD_URL}", channel: SLACK_CHANNEL, teamDomain: 'your-slack-team')
                }
            }
        }
        stage('Deploy to Jenkins Slave') {
            agent {
                node {
                    label "${JENKINS_SLAVE_LABEL}"
                }
            }
            steps {
                script {

                    // Remove existing container (if exists)
                    sh "docker rm -f ${DOCKER_IMAGE_NAME} || true"

                    // Load Docker image on Jenkins slave
                    sh "cd /home/ubuntu/GraduationProjectrepo"

                    // Run Docker container
                    sh "docker compose up -d"
                }
            }
        }
    }

    }
    post {
        failure {
            // Notify Slack about the failed build
            slackSend(color: '#FF0000', message: "Docker image build failed! \n${BUILD_URL}", channel: SLACK_CHANNEL)
        }
    }
