pipeline {
  agent any

  environment {
    // Docker Hub
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
    DOCKERHUB_USER = 'hsindhuja'

    BACKEND_IMAGE  = 'hsindhuja/calculator-backend'
    FRONTEND_IMAGE = 'hsindhuja/calculator-frontend'
    TAG = "${BUILD_NUMBER}"

    // Kubernetes
    NAMESPACE = 'default'
    BACKEND_DEPLOYMENT = 'calculator-backend'
    FRONTEND_DEPLOYMENT = 'calculator-frontend'
    BACKEND_CONTAINER = 'backend'
    FRONTEND_CONTAINER = 'frontend'
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main',
            url: 'https://github.com/Hema8368/k8s-calculator.git'
      }
    }

    stage('Build Docker Images') {
      steps {
        sh '''
          docker build -t $BACKEND_IMAGE:$TAG ./backend
          docker build -t $FRONTEND_IMAGE:$TAG ./frontend
        '''
      }
    }

    stage('Docker Hub Login & Push') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: "${DOCKERHUB_CREDENTIALS}",
            usernameVariable: 'DH_USER',
            passwordVariable: 'DH_PASS'
          )
        ]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push $BACKEND_IMAGE:$TAG
            docker push $FRONTEND_IMAGE:$TAG
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          kubectl apply -f k8s/

          kubectl set image deployment/$BACKEND_DEPLOYMENT \
            $BACKEND_CONTAINER=$BACKEND_IMAGE:$TAG -n $NAMESPACE

          kubectl set image deployment/$FRONTEND_DEPLOYMENT \
            $FRONTEND_CONTAINER=$FRONTEND_IMAGE:$TAG -n $NAMESPACE

          kubectl rollout status deployment/$BACKEND_DEPLOYMENT -n $NAMESPACE
          kubectl rollout status deployment/$FRONTEND_DEPLOYMENT -n $NAMESPACE

          kubectl get pods -n $NAMESPACE
          kubectl get svc -n $NAMESPACE
        '''
      }
    }
  }
}
