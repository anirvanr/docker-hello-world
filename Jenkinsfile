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

def helmPush(String chart_dir) {
    sh """
      /usr/local/bin/helm init --client-only
      /usr/local/bin/helm repo add chartmuseum https://chartmuseum.dynacommercelab.com/techm/megafon
      /usr/local/bin/helm push --context-path=/techm/megafon ${chart_dir} chartmuseum --username ${helm_user} --password ${helm_pass}
    """
}

def helmDeploy(Map args) {
    //configure helm client and confirm tiller process is installed
    if (args.dry_run) {
        println "Running dry-run deployment"
        sh "/usr/local/bin/helm upgrade --dry-run --debug --install ${args.name}-${args.env} ${args.chart_dir} -f ${args.chart_dir}/${args.env}-values.yaml --namespace=${args.env}"
    } else {
        println "Running deployment"
        sh "/usr/local/bin/helm upgrade --recreate-pods --install ${args.name}-${args.env} ${args.chart_dir} -f ${args.chart_dir}/${args.env}-values.yaml --namespace=${args.env}"
        echo "Application ${args.name} successfully deployed in ${args.env}. Use helm status ${args.name}-${args.env} to check"
    }
}

pipeline {
  agent any

  options {
    timeout(time: 15, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  environment {
    container_name = "hello-world"
    nexus_url = "dk.dynacommercelab.com"
    nexus_creds_id = "nexus-credentials"
    docker_image = "${nexus_url}/${container_name}"
    docker_tag = "1.0.3"
    // helm_repo_cred = credentials('helm-repo-creds')
  }

  parameters {
    choice(
        name: 'chartname',
        description: 'Which Chart do you want to build?',
        choices: 'dev\nhello-world'
    )
    string(
        name: 'version',
        description: 'Chart Version to deploy?',
        defaultValue: '0.1.0'
    )
    string(
        name: 'values',
        description: 'Any values to overwrite?',
        defaultValue: 'env.open.key1=val1,key2=val2'
    )
  }

  stages {
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
        withCredentials([usernamePassword(credentialsId: 'helm-repo-creds', passwordVariable: 'helm_pass', usernameVariable: 'helm_user')]) {
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
          helmPush(chart_dir)
          helmDeploy(
            dry_run     : false,
            name        : app_name,
            chart_dir   : chart_dir,
            tag         : build_tag,
            env         : deployEnv
            )
          }
        }
      }
    }

  stage('Manual Deployment'){
  when { expression { BRANCH_NAME ==~ /develop/ } }
  steps{
      script {
          if (params.version) { env.addVersion = "--version ${params.version}" }
          else { env.addVersion = ' '}
          if (params.values) { env.addValues = "--set-string ${params.values}"}
          else { env.addValues = ' '  }
          switch(params.chartname) {
              case 'dev':
                  env.namespace = "--namespace development";
              break;
              default:
                  env.namespace = '--namespace default';
          }
      }
      
      sh """
      helm repo update
      helm upgrade --install canary-${params.chartname} --atomic --wait --timeout 20s ${env.addVersion} ${env.addValues} ${env.namespace} --debug incubator/${params.chartname}
      """
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
  // post {
  //       always {

  //           echo 'slack notification....'
  //           script {
  //               if (BRANCH_NAME.startsWith('PR')) {
  //                   env.channel = '#jenkins-ci'
  //               } else {
  //                   env.channel = '#jenkins-ci'
  //               }
  //           }
  //       }

  //       failure {
  //           echo 'Failure!'
  //           slackSend channel: env.channel, color: "danger", message: "Build Failed: <${env.JOB_DISPLAY_URL}|${env.JOB_NAME}> <${env.RUN_DISPLAY_URL}|#${env.BUILD_NUMBER}>"
  //       }

  //       success {
  //           echo 'Success!'
  //           slackSend channel: env.channel, color: "good", message: "Build Passed: <${env.JOB_DISPLAY_URL}|${env.JOB_NAME}> <${env.RUN_DISPLAY_URL}|#${env.BUILD_NUMBER}>"
  //       }
  //   }
}