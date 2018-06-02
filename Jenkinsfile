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
            configFileProvider([configFile(fileId: 'f5f75a53-52f5-4fe3-bdff-9f0709d38940', replaceTokens: true, targetLocation: 'config', variable: 'configfile')]) {        
                sh '''
                    sh 'cat $configfile'
                    # apt-get update && apt-get -y install curl
                    # curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl
                    ls -lah
                '''
        }
      }
    }
  }
}
