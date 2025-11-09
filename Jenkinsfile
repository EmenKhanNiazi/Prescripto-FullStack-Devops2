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
        withCredentials([string(credentialsId: 'VITE_BACKEND_URL', variable: 'VITE_BACKEND_URL')]) {
          sh '''
            echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > clientside/.env
            echo "VITE_BACKEND_URL=${VITE_BACKEND_URL}" > admin/.env
            echo "Wrote clientside/.env and admin/.env:"
            # do not print the secret in production; this is for debug
            cat clientside/.env || true
            cat admin/.env || true
          '''
        }
      }
    }

    stage('CI Build (containerized)') {
      steps {
        // ensure no leftover CI containers using docker/compose image
        sh '''
          # run docker-compose down using compose container
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v "$(pwd)":/workdir -w /workdir \
            docker/compose:latest -f ${CI_COMPOSE} down --volumes --remove-orphans || true
        '''
        // run build (abort when containers exit)
        sh '''
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v "$(pwd)":/workdir -w /workdir \
            docker/compose:latest -f ${CI_COMPOSE} up --build --abort-on-container-exit --remove-orphans
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
      // cleanup CI containers using docker/compose container and remove transient .env files
      sh '''
        docker run --rm \
          -v /var/run/docker.sock:/var/run/docker.sock \
          -v "$(pwd)":/workdir -w /workdir \
          docker/compose:latest -f ${CI_COMPOSE} down --volumes --remove-orphans || true
        rm -f clientside/.env admin/.env || true
      '''
    }
  }
}
