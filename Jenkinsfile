pipeline {
  agent { label 'docker-agent' }
  stages {
    stage('Build') {
      when {
        branch 'master'
      }
      steps {
        sh "docker build -t anirvan/hello-world:${env.BRANCH_NAME} ."
      }
    }
    stage('Publish') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([credentialsId: 'docker-hub-credentials']) {
          sh "docker push anirvan/hello-world:${env.BRANCH_NAME}"
        }

      }
    }
  }
}
