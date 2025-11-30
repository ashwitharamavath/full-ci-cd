pipeline {
  agent any

  environment {
    // Your Docker Hub repository
    DOCKER_HUB_REPO = 'vyshnavi525/my-app'
    // Tag images like v1, v2, v3...
    IMAGE_TAG = "v${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        echo "📥 Checking out source code from Git..."
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        echo "🐳 Building Docker image..."
        bat """
        docker build -t %DOCKER_HUB_REPO%:%IMAGE_TAG% .
        """
      }
    }

    stage('Login & Push to Docker Hub') {
      steps {
        echo "🔐 Logging in and pushing image to Docker Hub..."
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub',       // Jenkins credential ID
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          bat """
          echo %PASS% | docker login -u %USER% --password-stdin

          REM Tag image with 'latest' also
          docker tag %DOCKER_HUB_REPO%:%IMAGE_TAG% %DOCKER_HUB_REPO%:latest

          REM Push versioned tag and latest tag
          docker push %DOCKER_HUB_REPO%:%IMAGE_TAG%
          docker push %DOCKER_HUB_REPO%:latest

          docker logout
          """
        }
      }
    }

    stage('Deploy to Kubernetes (Minikube)') {
      steps {
        echo "🚀 Deploying to Kubernetes (Minikube)..."
        bat """
        REM Point kubectl to your kubeconfig (change 'ganga' if your username is different)
        set KUBECONFIG=C:\\Users\\ganga\\.kube\\config

        REM Make sure we are using the minikube context
        kubectl config use-context minikube

        REM Apply deployment and service (skip strict validation to avoid openapi auth issue)
        kubectl apply -f deployment.yaml --validate=false
        kubectl apply -f service.yaml --validate=false

        REM Show pods and services for verification
        kubectl get pods
        kubectl get svc
        """
      }
    }
  }

  post {
    success {
      echo "✅ Docker image built and pushed: ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}"
      echo "✅ Deployed to Kubernetes (Minikube) using deployment.yaml and service.yaml"
    }
    failure {
      echo "❌ Pipeline failed. Check the stage logs above for details."
    }
  }
}
