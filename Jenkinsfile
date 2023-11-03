pipeline {
  agent any
  triggers {
    githubPush()
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    KUBECONFIG = credentials('minikubeconfig')    
    DOCKERHUB_CREDENTIALS = credentials('evandjefie-dockerhub')
    ORGANIZATION_NAME = 'evandjefie'
    DOCKERHUB_CREDENTIALS_USR = 'evandjefie'
    APP_NAME = 'my-static-portfolio'
    APP_VERSION = '1.0.1'
    REPO_URL = 'https://github.com/evandjefie/my-static-portfolio.git'
  }

  stages {
    stage('Clone App') {
      steps {
        sh 'git clone $REPO_URL app'
      }
    }    
    stage('Build and Tag') {
      steps {
        sh 'cd app && docker build -t ${ORGANIZATION_NAME}/${APP_NAME}:${APP_VERSION} .'
      }
    }
    stage('Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push ${ORGANIZATION_NAME}/${APP_NAME}:${APP_VERSION}'
      }
    }
    stage('Deploy App') {
      steps {
        sh '''
          sleep 10s
          cd app
          kubectl apply -f k8s/static-app-dpl.yaml
          kubectl apply -f k8s/static-app-svc.yaml
          kubectl get all -A   
        '''
      }
    }    
  }
  post {
    always {        
      sh 'docker logout'
    }
    failure {
        deleteDir()
    }
  }
}