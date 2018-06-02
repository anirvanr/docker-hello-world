#!groovy

pipeline {
  agent { label 'docker-agent' }
  
  stages {    
    stage('Image Build') {
      when {
        branch 'master'
      }
      steps {
        sh "git rev-parse --short HEAD > .git/commit-id"                        
        commit_id = readFile('.git/commit-id')
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
