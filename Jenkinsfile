#!groovy

pipeline {
  agent any

  environment {
        NEXUS_URL = 'dk.dynacommercelab.com'
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
        DOCKER_IMAGE = 'dk.dynacommercelab.com/hello-world'
        TAG = 'latest'
    }
  
  stages {
    stage('Build') {
      steps {
        sh 'docker build -f "Dockerfile" -t ${DOCKER_IMAGE}:${TAG} .'
      }
    }
    stage('Publish') {
      when {
        branch 'master'
      }
      steps {
        sh 'docker push ${DOCKER_IMAGE}:${TAG}'
      }
    }
  }
}