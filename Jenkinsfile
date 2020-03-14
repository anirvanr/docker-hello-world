#!groovy

pipeline {
  agent any

  environment {
    NAME = 'hello-world'
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
    stage('Deploy development') {
        when {
            expression { BRANCH_NAME ==~ /develop/ }
        }
        steps {
          withDockerRegistry([ credentialsId: "${NEXUS_CREDENTIAL_ID}", url: "${NEXUS_URL}" ]){
          sh '''
          IMAGE_HASH="$(docker pull $DOCKER_IMAGE | grep 'Digest: ' | sed 's/Digest: //')"
          '''
          withCredentials([kubeconfigContent(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
          sh '''
          echo $KUBECONFIG_CONTENT
          /usr/local/bin/kubectl --namespace=development set image deployment/${NAME} ${NAME}=${DOCKER_IMAGE}@${IMAGE_HASH} --record
          '''
          }
        }
      }
    }
    stage('Deploy production') {
      when {
          expression { BRANCH_NAME ==~ /master/ }
      }
      steps {
          withCredentials([kubeconfigContent(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
          sh '''
          export KUBECONFIG=${PWD}/config
          /usr/local/bin/kubectl --namespace=production set image deployment/${NAME} ${NAME}=${DOCKER_IMAGE}:${TAG} --record
          '''
        }
      }
    }
  }
}