#!groovy

pipeline {
  agent any

  environment {
    NEXUS_URL = 'https://dk.dynacommercelab.com'
    NEXUS_CREDENTIAL_ID = "nexus-credentials"
    DOCKER_IMAGE = 'dk.dynacommercelab.com/hello-world'
    TAG = '1.0.1'
    }
  
  stages {
    stage('Dockerize') {
      steps {
        echo 'Dockerizing...'
          withDockerRegistry([ credentialsId: "${NEXUS_CREDENTIAL_ID}", url: "${NEXUS_URL}" ]){
          sh '''
          if [[ "$BRANCH_NAME" =~ master ]]
          then
            docker build -f "Dockerfile" -t $DOCKER_IMAGE:$TAG .
            docker push $DOCKER_IMAGE:$TAG
          elif [[ "$BRANCH_NAME" =~ develop ]]
          then
            docker build -f "Dockerfile" -t $DOCKER_IMAGE .
            docker push $DOCKER_IMAGE
          else
            echo 'Do that only on master or develop branch'
          fi
          '''
        }
      }
    }
  }
}