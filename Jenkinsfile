pipeline {
  agent any

  options { skipDefaultCheckout(true) }

  environment {
    DOCKER_IMAGE = "ermek1988/manual-app"
    BUILD_TAG    = "build-${BUILD_NUMBER}"
    COMMIT_SHA   = ""   // заполним после checkout
  }

  stages {
    stage('Checkout') {
      steps {
        echo "Cloning repository..."
        checkout scm
        script {
          COMMIT_SHA = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
          env.COMMIT_SHA = COMMIT_SHA
        }
      }
    }

    stage('Verify Docker') {
      steps {
        sh '''
          echo "User: $(whoami)"
          echo "PATH=$PATH"
          command -v docker
          docker --version
          ls -l /var/run/docker.sock || true
        '''
      }
    }

    stage('Install Dependencies') {
      steps {
        echo "Installing npm dependencies (via Docker node:20)..."
        sh '''
          docker run --rm -v "$PWD":/app -w /app node:20 npm ci
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        echo "Building Docker image..."
        sh '''
          docker build -t ${DOCKER_IMAGE}:${BUILD_TAG} .


exit
echo "Dockerfile.jenkins" >> .gitignore
