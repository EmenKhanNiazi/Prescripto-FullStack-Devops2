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
        // NOTE: set the credentialsId to the Secret Text you added in Jenkins (VITE_BACKEND_URL)
        withCredentials([string(credentialsId: 'VITE_BACKEND_URL', variable: 'VITE_BACKEND_URL')]) {
          sh '''
            # create .env files inside the workspace (these are transient)
            echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > clientside/.env
            echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > admin/.env
            echo "Wrote clientside/.env and admin/.env"
            # do not print the secret in production; kept minimal here
            ls -la clientside admin || true
          '''
        }
      }
    }

    stage('CI Build (containerized)') {
      steps {
        // debug workspace to confirm files
        sh 'pwd; ls -la . || true'

        // run docker-compose down using official docker/compose container (absolute path)
        sh '''
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v "$(pwd)":/workdir -w /workdir \
            docker/compose:latest -f /workdir/${CI_COMPOSE} down --volumes --remove-orphans || true
        '''

        // run docker-compose up to build inside node containers (abort when containers exit)
        sh '''
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v "$(pwd)":/workdir -w /workdir \
            docker/compose:latest -f /workdir/${CI_COMPOSE} up --build --abort-on-container-exit --remove-orphans
        '''
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'clientside/dist/**, admin/dist/**', allowEmptyArchive: true
      }
    }
  }

  post {
    always {
      // ensure clean state and remove transient .env files
      sh '''
        docker run --rm \
          -v /var/run/docker.sock:/var/run/docker.sock \
          -v "$(pwd)":/workdir -w /workdir \
          docker/compose:latest -f /workdir/${CI_COMPOSE} down --volumes --remove-orphans || true
        rm -f clientside/.env admin/.env || true
      '''
    }
  }
}
