pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "krishnan14"
        DOCKER_CREDENTIALS = "DOCKER_CREDENTIALS"  // Jenkins credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/krishnan1412/MernDocker.git'
            }
        }

        stage('Install Docker') {
            steps {
                script {
                    sh "sudo apt update"
                    sh "sudo apt install -y docker.io docker-compose-plugin"
                    sh "docker --version"
                    sh "docker compose version"
                }
            }
        }
        stage('add privilage to docker') {
            steps {
                script {
                    sh "sudo usermod -aG docker jenkins"
                    sh "sudo systemctl restart jenkins"
                }
            }
        }

        stage('Build & Tag Images') {
            steps {
                script {
                    // Build all services defined in docker-compose.yaml
                    sh "docker compose build"

                    // Tag each image with DockerHub registry prefix
                    def services = sh(script: "docker compose config --services", returnStdout: true).trim().split('\n')
                    for (svc in services) {
                        sh "docker tag ${svc}:latest ${DOCKER_REGISTRY}/${svc}:latest"
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        def services = sh(script: "docker compose config --services", returnStdout: true).trim().split('\n')
                        for (svc in services) {
                            sh "docker push ${DOCKER_REGISTRY}/${svc}:latest"
                        }
                    }
                }
            }
        }

        stage('Deploy Updated Stack') {
            steps {
                script {
                    // Pull latest images from DockerHub
                    sh "docker compose pull"
                    // Recreate containers with latest images
                    sh "docker compose up -d"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Images built, pushed, and stack redeployed."
        }
    }
}

