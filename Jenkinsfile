#!groovy

pipeline {
  agent { 
    label 'docker-agent' 
  }  
  environment {
     commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
  }
  stages { 
    stage('Image Build') {
      when {
        branch 'master'
      }     
      steps {
        sh "docker build -t anirvan/hello-world:${commit_id} ."
      }
    }
    stage('Push to Docker Registry') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} --password-stdin ${env.dockerHubPassword}"
          sh "docker push anirvan/hello-world:${commit_id}"
        }
      }
    }
  }
}
