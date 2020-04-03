#!groovy

def deployEnv

def kubectlTest() {
    // Test that kubectl can correctly communication with the Kubernetes API
    echo "running kubectl test"
    sh "/usr/local/bin/kubectl get nodes"
}

def helmLint(String chart_dir, String deployEnv) {
    // lint helm chart
    sh "/usr/local/bin/helm lint ${chart_dir} -f ${chart_dir}/${deployEnv}-values.yaml"
}

def helmDeploy(Map args) {
    //configure helm client and confirm tiller process is installed
    if (args.dry_run) {
        println "Running dry-run deployment"
        sh "/usr/local/bin/helm upgrade --dry-run --debug --install ${args.name}-${args.env} ${args.chart_dir} -f ${args.chart_dir}/${args.env}-values.yaml --namespace=${args.env}"
    } else {
        println "Running deployment"
        sh "/usr/local/bin/helm upgrade --install ${args.name}-${args.env} ${args.chart_dir} -f ${args.chart_dir}/${args.env}-values.yaml --namespace=${args.env}"
        echo "Application ${args.name} successfully deployed in ${args.env}. Use helm status ${args.name} to check"
    }
}

pipeline {
  agent any

  environment {
    container_name = "hello-world"
    nexus_url = "dk.dynacommercelab.com"
    nexus_creds_id = "nexus-credentials"
    docker_image = "${nexus_url}/${container_name}"
    docker_tag = "1.0.3"
    }
  stages {
    // stage('read') {
    //     steps {
    //       script {
    //           def data = readFile(file: 'config.json')
    //           println(data)
    //           def inputFile = readFile('config.json')
    //           def config = new groovy.json.JsonSlurperClassic().parseText(inputFile)
    //           println "pipeline config ==> ${config}"
    //       }
    //     }
    //   }
    // stage('Dockerize') {
    //   steps {
    //     echo 'Dockerizing...'
    //       withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "https://${nexus_url}" ]){
    //       sh '''
    //       if [[ ${BRANCH_NAME} =~ master ]]
    //       then
    //         docker build -f "Dockerfile" -t ${docker_image}:${docker_tag} .
    //         docker push ${docker_image}:${docker_tag} || { >&2 echo "Failed to push build_tag '${docker_tag}' image ${docker_image}"; exit 1; }
    //       elif [[ ${BRANCH_NAME} =~ develop ]]
    //       then
    //         docker build -f "Dockerfile" -t ${docker_image}:latest .
    //         docker push ${docker_image} || { >&2 echo "Failed to push build_tag 'latest' image ${docker_image}"; exit 1; }
    //       else
    //         echo 'Do that only on master or develop branch'
    //       fi
    //       '''
    //     }
    //   }
    // }
    stage('Dockerize') {
      steps {
        echo 'Dockerizing...'
          withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "https://${nexus_url}" ]){
          script {
            if ( env.BRANCH_NAME == "develop" ){
            sh '''
              docker build -f "Dockerfile" -t ${docker_image}:latest .
              docker push ${docker_image} || { >&2 echo "Failed to push build_tag 'latest' image ${docker_image}"; exit 1; }
            '''
            } else if ( env.BRANCH_NAME == "master" ){
            sh '''
              docker build -f "Dockerfile" -t ${docker_image}:${docker_tag} .
              docker push ${docker_image}:${docker_tag} || { >&2 echo "Failed to push build_tag '${docker_tag}' image ${docker_image}"; exit 1; }
            '''
            } else{
              deployEnv = "none"
              error "Building unknown branch"
            }
          }
        }
      }
    }
    stage ('helm deploy') {
      steps{
        script {
          if ( env.BRANCH_NAME == "develop" ){
            deployEnv = "development"
          } else if ( env.BRANCH_NAME == "master" ){
            deployEnv = "production"
          } else{
            deployEnv = "none"
            error "Building unknown branch"
          }
          def pwd = pwd()
          def app_name = "${container_name}"
          def chart_dir = "${pwd}/charts/${container_name}"
          // run helm chart linter
          helmLint(chart_dir,deployEnv)
          kubectlTest()
          helmDeploy(
          dry_run       : false,
          name          : app_name,
          chart_dir     : chart_dir,
          tag           : build_tag,
          env           : deployEnv
          )
        }
      }
    }
    // stage('Deploy development') {
    //     when {
    //       expression { BRANCH_NAME ==~ /develop/ }
    //     }
    //     steps {
    //       withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "https://${nexus_url}" ]){
    //       sh '''
    //       docker_image_hash="$(docker pull ${docker_image} | grep 'Digest: ' | sed 's/Digest: //')"
    //       /usr/local/bin/kubectl --namespace=development set image deployment/${container_name} ${container_name}=${docker_image}@${docker_image_hash} --record
    //       '''
    //     }
    //   }
    // }
    // stage('Deploy production') {
    //     when {
    //       expression { BRANCH_NAME ==~ /master/ }
    //     }
    //     steps {
    //       withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "https://${nexus_url}" ]){
    //       sh '''
    //       echo ${kubectlTest}
    //       docker_image_hash="$(docker pull $docker_image:${build_tag} | grep 'Digest: ' | sed 's/Digest: //')"
    //       /usr/local/bin/kubectl --namespace=production set image deployment/${container_name} ${container_name}=${docker_image}@${docker_image_hash} --record
    //       '''
    //     }
    //   }
    // }
    // stage('analyze') {
    //       when {
    //       branch 'master'
    //     }
    //     steps {
    //       sh 'echo "dk.dynacommercelab.com/hello-world:latest `pwd`/Dockerfile" > anchore_images'
    //       anchore name: 'anchore_images'
    //       }
    //     }
    // stage('Publish') {
    //     when {
    //       branch 'master'
    //     }
    //     environment {
    //       DOCKER_IMAGE_MF = "dkmf.dynacommercelab.com/${NAME}"
    //       nexus_url_MF = "https://dkmf.dynacommercelab.com"
    //     }
    //     steps {
    //       script {
    //           timeout(time: 1, unit: 'MINUTES') {
    //           env.IMAGE_PUSH = input message: 'User input required', ok: 'Continue',
    //           parameters: [choice(name: 'Upload docker image', choices: 'yes\nno', description: '')]
    //           }
    //       withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "${nexus_url_MF}" ]){
    //       sh '''
    //       if [[ ${IMAGE_PUSH} == 'yes' ]]
    //       then
    //           docker tag ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE_MF}:${TAG}
    //           docker push ${DOCKER_IMAGE_MF}:${TAG}
    //       else
    //         echo "don't do that"
    //       fi
    //       '''
    //       } 
    //     } 
    //   }
    // }
  }
}