#!groovy

pipeline {
  agent { label 'docker-agent' }
  
  stages {    
    stage('Image Build') {
      when {
        branch 'master'
      }
      steps {
        sh "docker build -t anirvan/hello-world:${env.BUILD_ID} ."
      }
    }
    stage('Push to Docker Registry') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          sh "docker push anirvan/hello-world:${env.BUILD_ID}"
        }
      }
    }
  }
}
