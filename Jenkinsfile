#!groovy

pipeline {
  agent { 
    label 'docker-agent' 
  }  
  environment {
     commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
     image = 'hello-world'
     organization = 'anirvan'
     HELM_REPO_CREDS = credentials('helm-repo-creds')
  }
  stages { 
    stage('Image Build') {
      when {
        branch 'master'
      }     
      steps {
        sh "docker build -t ${organization}/${image}:${commit_id} ."
      }
    }
    stage('Push to Docker Registry') {
      when {
        branch 'master'
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          sh "docker push ${env.dockerHubUser}/${image}:${commit_id}"
        }
      }
    }
    stage('Deploy release') {
       steps {
            configFileProvider([configFile(fileId: 'f5f75a53-52f5-4fe3-bdff-9f0709d38940', replaceTokens: true, targetLocation: 'config', variable: 'configfile')]) {        
                sh '''
                    export KUBECONFIG=${PWD}/config
                    apt-get update && apt-get -y install curl
                    #curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl
                    #./kubectl get deployment hello-world -n preview -o wide
                    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
                    helm init --client-only
                    helm repo add chartmuseum https://helmcharts.dynacommercelab.com --username ${HELM_REPO_CREDS_USR} --password ${HELM_REPO_CREDS_PSW}
                    
                    helm plugin install https://github.com/chartmuseum/helm-push
                    helm push ${PWD}/charts/hello-world/ --version="$commit_id" chartmuseum
                    helm repo update
                    
                    helm search chartmuseum/ -l
                    echo "upgrade/install a release to a new version of a chart"
                    helm upgrade hello-world chartmuseum/hello-world --version=${commit_id} --set image.tag=${commit_id} --install --namespace preview
                    helm ls -a
                    helm history hello-world
                    
                  '''
        }
      }
    }
  }
}
