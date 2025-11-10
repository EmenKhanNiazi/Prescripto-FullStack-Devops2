// Jenkinsfile (Declarative Pipeline)

pipeline {
    agent any

    options {
        timeout(time: 15, unit: 'MINUTES') 
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Cloning repository from GitHub...'
                checkout scm
            }
        }

        stage('Stop Existing Containers') {
            steps {
                echo 'Stopping and removing any previous containers to meet "down initially" requirement...'
                // CRITICAL: Ensures the environment is down initially
                sh 'docker-compose -f docker-compose-part2.yml down --remove-orphans'
            }
        }

        stage('Containerized Up') {
            steps {
                echo 'Launching containerized application using docker-compose-part2.yml...'
                // 'up -d' launches the app in detached mode
                sh 'docker-compose -f docker-compose-part2.yml up -d'
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Checking container status...'
                sleep 10 
                sh 'docker-compose -f docker-compose-part2.yml ps'
                echo "Deployment Complete. Access at http://35.173.129.203:5174"
            }
        }
    }
}