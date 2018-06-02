#!groovy

pipeline {
  agent { 
    label 'docker-agent' 
  }  
  environment {
     commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
     image = 'hello-world'
     organization = 'anirvan'
  }
  stages { 
    stage('Image Build') {
      when {
        branch 'master'
      }     
      steps {
        sh "docker build -t ${env.organization}/${env.image}:${env.commit_id} ."
      }
    }
    stage('Push to Docker Registry') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          sh "docker push ${env.dockerHubUser}/${env.image}:${env.commit_id}"
        }
      }
    }
    stage('deploy') {
            // sh "ansible-playbook -i ./ansible/hosts ./ansible/deploy.yml"
       steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps"
                    ls -lah
                '''
      }
    }
  }
}
