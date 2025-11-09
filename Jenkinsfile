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
        // read secret credential and write .env files for client and admin
        withCredentials([string(credentialsId: 'VITE_BACKEND_URL', variable: 'VITE_BACKEND_URL')]) {
          // create .env files (safe overwrite) — these are only on the CI host workspace
          sh '''
            echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > clientside/.env
            echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > admin/.env
            echo "Wrote clientside/.env and admin/.env:"
            cat clientside/.env || true
            cat admin/.env || true
          '''
        }
      }
    }

    stage('CI Build (containerized)') {
      steps {
        // ensure no leftover CI containers
        sh "docker compose -f ${CI_COMPOSE} down --volumes --remove-orphans || true"
        // bring up CI compose which will run npm install/build inside node containers
        sh "docker compose -f ${CI_COMPOSE} up --build --abort-on-container-exit --remove-orphans"
      }
    }

    stage('Archive') {
      steps {
        // adjust paths if your builds place artifacts elsewhere
        archiveArtifacts artifacts: 'clientside/dist/**, admin/dist/**', allowEmptyArchive: true
      }
    }
  }

  post {
    always {
      // cleanup CI containers and remove the transient .env files for safety
      sh "docker compose -f ${CI_COMPOSE} down --volumes --remove-orphans || true"
      sh "rm -f clientside/.env admin/.env || true"
    }
  }
}
