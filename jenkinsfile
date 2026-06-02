pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "harryskouam" // replace this with your docker-id
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
COMPOSE_PROJECT_NAME = "jenkinsexam" // pin compose project name so built image names are deterministic
}
agent any // Jenkins will be able to select all available agents
stages {
  stage(' Docker Build'){ // docker build image stage
    steps {
      script {
      sh '''
        docker rm -f jenkins || true
        docker compose down || true
        docker compose build
      sleep 6
      '''
      }
    }
  }
  stage('Docker run'){ // run container from our builded image
    steps {
      script {
      sh '''
      docker compose up -d
      sleep 10
      '''
      }
    }
  }
  stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
    steps {
      script {
      sh '''
      curl -f http://localhost:8080/api/v1/movies/docs
      curl -f http://localhost:8080/api/v1/casts/docs
      '''
      }
    }
  }
  stage('Docker Push'){ //we pass the built image to our docker hub account
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
    }
    steps {
      script {
        sh '''
        # Compose v2 builds images named ${COMPOSE_PROJECT_NAME}-<service>
        docker tag ${COMPOSE_PROJECT_NAME}-movie_service \
        $DOCKER_ID/movie_service:$DOCKER_TAG

        docker tag ${COMPOSE_PROJECT_NAME}-cast_service \
        $DOCKER_ID/cast_service:$DOCKER_TAG

        docker login -u $DOCKER_ID -p $DOCKER_PASS

        docker push $DOCKER_ID/movie_service:$DOCKER_TAG
        docker push $DOCKER_ID/cast_service:$DOCKER_TAG
        '''
      }
    }
  }
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
        --namespace dev \
        --set movieService.image.tag=$DOCKER_TAG \
        --set castService.image.tag=$DOCKER_TAG
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
        --set movieService.image.tag=$DOCKER_TAG \
        --set castService.image.tag=$DOCKER_TAG
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
            --set movieService.image.tag=$DOCKER_TAG \
            --set castService.image.tag=$DOCKER_TAG
          '''
        }
      }
    }
  }
}