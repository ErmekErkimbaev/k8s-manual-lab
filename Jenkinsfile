pipeline {
  agent any

  options { skipDefaultCheckout(true) }

  environment {
    DOCKER_IMAGE = "aisalkyn85/manual-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        echo "Cloning repository..."
        checkout scm
      }
    }

    // 👇 ВОТ ЭТОТ STAGE ВСТАВЛЯЕМ МЕЖДУ Checkout и Install Dependencies
    stage('Debug Environment') {
      steps {
        sh '''
          echo "=== DEBUG ENV ==="
          whoami
          uname -a
          echo "PATH=$PATH"
          which docker || true
          ls -l /usr/bin/docker /usr/local/bin/docker 2>/dev/null || true
          docker --version || true
          ls -l /var/run/docker.sock || true
          echo "=== END DEBUG ==="
        '''
      }
    }

    stage('Install Dependencies') {
      steps {
        echo "Installing npm dependencies (via Docker)..."
        sh '''
          docker --version
          docker run --rm -v "$PWD":/app -w /app node:20 npm ci
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        echo "Building Docker image..."
        sh 'docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .'
      }
    }

    stage('Login to DockerHub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
        }
      }
    }

    stage('Push Image') {
      steps {
        echo "Pushing image to DockerHub..."
        sh 'docker push ${DOCKER_IMAGE}:${IMAGE_TAG}'
      }
    }
  }

  post {
    success { echo "CI Pipeline completed successfully!" }
    failure { echo "CI Pipeline failed!" }
    always  { echo "Pipeline finished." }
  }
}
