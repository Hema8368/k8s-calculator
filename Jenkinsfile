pipeline {
  agent any

  environment {
    DOCKERHUB_CRED = 'dockerhub-creds'
    DOCKERHUB_USER = 'hsindhuja'

    BACKEND_IMAGE  = "${DOCKERHUB_USER}/calculator-backend"
    FRONTEND_IMAGE = "${DOCKERHUB_USER}/calculator-frontend"
    TAG = "${BUILD_NUMBER}"

    K8S_NAMESPACE = "default"
    BACKEND_DEPLOYMENT = "calculator-backend"
    FRONTEND_DEPLOYMENT = "calculator-frontend"

    BACKEND_CONTAINER = "backend"
    FRONTEND_CONTAINER = "frontend"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build Docker Images') {
      steps {
        sh '''
          docker build -t ${BACKEND_IMAGE}:${TAG} ./backend
          docker build -t ${FRONTEND_IMAGE}:${TAG} ./frontend
        '''
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CRED}", usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push ${BACKEND_IMAGE}:${TAG}
            docker push ${FRONTEND_IMAGE}:${TAG}
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          export KUBECONFIG=/var/jenkins_home/.kube/config

          kubectl apply -n ${K8S_NAMESPACE} -f k8s/ --validate=false

          kubectl set image -n ${K8S_NAMESPACE} deployment/${BACKEND_DEPLOYMENT} \
            ${BACKEND_CONTAINER}=${BACKEND_IMAGE}:${TAG}

          kubectl set image -n ${K8S_NAMESPACE} deployment/${FRONTEND_DEPLOYMENT} \
            ${FRONTEND_CONTAINER}=${FRONTEND_IMAGE}:${TAG}

          kubectl rollout status -n ${K8S_NAMESPACE} deployment/${BACKEND_DEPLOYMENT} --timeout=180s
          kubectl rollout status -n ${K8S_NAMESPACE} deployment/${FRONTEND_DEPLOYMENT} --timeout=180s

          kubectl get pods -n ${K8S_NAMESPACE} -o wide
          kubectl get svc -n ${K8S_NAMESPACE}
        '''
      }
    }
  }
}
