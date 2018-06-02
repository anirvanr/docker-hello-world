#!groovy

pipeline {
  agent { label 'docker-agent' }
  
  stages {    
    stage('Image Build') {
      when {
        branch 'master'
      }     
      steps {
        commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        sh "docker build -t anirvan/hello-world:${commit_id} ."
      }
    }
    stage('Push to Docker Registry') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          sh "docker push anirvan/hello-world:${commit_id}"
        }
      }
    }
  }
}
