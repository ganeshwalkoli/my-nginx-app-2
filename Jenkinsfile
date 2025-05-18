pipeline {
  agent {
    docker {
      image 'ganeshwalkoli/ubuntu-build-tool:v0.2'
      args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  environment {
    IMAGE_NAME = "test-nginx-app"
    IMAGE_TAG = "v1.0.${BUILD_NUMBER}"
    REGISTRY = "docker.io/ganeshwalkoli"
    FULL_IMAGE = "${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/ganeshwalkoli/my-nginx-app-2.git'
      }
    }

    stage('Build Angular App') {
      steps {
        sh 'npm install'
        sh 'ng build --configuration production'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $FULL_IMAGE .'
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-ganesh', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push $FULL_IMAGE
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          kubectl set image deployment/nginx-app nginx-app=$FULL_IMAGE --namespace=default || kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
        '''
      }
    }
  }
}

