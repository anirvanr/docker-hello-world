#!groovy

pipeline {
  agent { label 'docker-agent' }
  
   stages {
    stage('Build') {
      when {
        branch 'master'
      }
      steps {
        sh "docker build -t anirvan/hello-world:${GIT_SHA} ."
      }
    }
    stage('Publish') {
      when {
        branch 'master'
      }
      steps {
        withDockerRegistry([credentialsId: 'docker-hub-credentials', url: 'https://hub.docker.com']) {
          sh "docker push hub.docker.com/anirvan/hello-world:${GIT_SHA}"
        }
      }
    }
  }
}
