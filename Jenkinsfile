pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "harryskouam" // docker hub account hosting the pre-built images
}
agent any // Jenkins will be able to select all available agents
stages {
  stage('Deploiement en dev'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      cat $KUBECONFIG > .kube/config
      helm upgrade --install app ./charts \
        --kubeconfig .kube/config \
        --namespace dev
      '''
      }
    }
  }
  stage('Deploiement en staging'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      cat $KUBECONFIG > .kube/config
      helm upgrade --install app ./charts \
        --kubeconfig .kube/config \
        --namespace staging \
        --set nginx.service.nodePort=30081
      '''
      }
    }
  }
  stage('Deploiement en prod'){
    environment {
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {
        // Create an Approval Button with a timeout of 15minutes.
        // this require a manuel validation in order to deploy on production environment
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          rm -Rf .kube
          mkdir .kube
          cat $KUBECONFIG > .kube/config
          helm upgrade --install app ./charts \
            --kubeconfig .kube/config \
            --namespace prod \
            --set nginx.service.nodePort=30082
          '''
        }
      }
    }
  }
}
