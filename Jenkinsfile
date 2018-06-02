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
       steps {
            // Kubectl setup
            configFileProvider([configFile(fileId: 'f5f75a53-52f5-4fe3-bdff-9f0709d38940', replaceTokens: true, targetLocation: 'config', variable: 'configfile')]) {        
                sh '''
                    export KUBECONFIG=$PWD/config
                    apt-get update && apt-get -y install curl
                    curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl
                    ./kubectl get pods --all-namespaces       
                '''
           // Helm setup
               sh '''
                    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
                    helm init --client-only
                    helm repo add chartmuseum https://helmcharts.dynacommercelab.com --username admin --password C1@oHCR$
                    helm repo update
                    helm search chartmuseum/ -l
               '''
        }
      }
    }
  }
}
