pipeline {
    agent any
    environment {
        CI_COMPOSE = 'docker-compose.ci.yml'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Inject env for frontends') {
            steps {
                withCredentials([string(credentialsId: 'vite-backend-url', variable: 'VITE_BACKEND_URL')]) {
                    sh '''
                    echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > clientside/.env
                    echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > admin/.env
                    echo "Wrote clientside/.env and admin/.env"
                    ls -la clientside admin
                    '''
                }
            }
        }

        stage('CI Build (docker-compose native)') {
            steps {
                sh '''
                pwd
                ls -la .
                docker-compose -f ${CI_COMPOSE} down --volumes --remove-orphans || true
                docker-compose -f ${CI_COMPOSE} up --build --abort-on-container-exit --remove-orphans
                '''
            }
        }

        stage('Archive') {
            steps {
                echo 'Build done.'
            }
        }
    }

    post {
        always {
            sh '''
            docker-compose -f ${CI_COMPOSE} down --volumes --remove-orphans || true
            rm -f clientside/.env admin/.env
            '''
        }
    }
}
