#!groovy

  def kubectlTest() {
    // Test that kubectl can correctly communication with the Kubernetes API
    echo "running kubectl test"
    sh "kubectl get nodes"

}

pipeline {
  agent any

  environment {
    container_name = "hello-world"
    nexus_url = "dk.dynacommercelab.com"
    nexus_creds_id = "nexus-credentials"
    docker_image = "${nexus_url}/${container_name}"
    build_tag = "1.0.3"
    }


  stages {
    stage('read') {
        steps {
            script {
                def data = readFile(file: 'config.json')
                println(data)
    def inputFile = readFile('config.json')
    def config = new groovy.json.JsonSlurperClassic().parseText(inputFile)
    println "pipeline config ==> ${config}"
            }
        }
      }
    stage('Dockerize') {
      steps {
        echo 'Dockerizing...'
          withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "https://${nexus_url}" ]){
          sh '''
          if [[ ${branch_name} =~ master ]]
          then
            docker build -f "Dockerfile" -t ${docker_image}:${build_tag} .
            docker push ${docker_image}:${build_tag} || { >&2 echo "Failed to push build_tag '${build_tag}' image ${docker_image}"; exit 1; }
          elif [[ ${branch_name} =~ develop ]]
          then
            docker build -f "Dockerfile" -t ${docker_image}:latest .
            docker push ${docker_image} || { >&2 echo "Failed to push build_tag 'latest' image ${docker_image}"; exit 1; }
          else
            echo 'Do that only on master or develop branch'
          fi
          '''
        }
      }
    }

    stage('Deploy development') {
        when {
          expression { branch_name ==~ /develop/ }
        }
        steps {
          withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "https://${nexus_url}" ]){
          sh '''
          IMAGE_HASH="$(docker pull ${docker_image} | grep 'Digest: ' | sed 's/Digest: //')"
          /usr/local/bin/kubectl --namespace=development set image deployment/${container_name} ${container_name}=${docker_image}@${IMAGE_HASH} --record
          '''
        }
      }
    }
    stage('Deploy production') {
        when {
          expression { branch_name ==~ /master/ }
        }
        steps {
          withDockerRegistry([ credentialsId: "${nexus_creds_id}", url: "https://${nexus_url}" ]){
          sh '''
          echo ${kubectlTest}
          IMAGE_HASH="$(docker pull $docker_image:${build_tag} | grep 'Digest: ' | sed 's/Digest: //')"
          /usr/local/bin/kubectl --namespace=production set image deployment/${container_name} ${container_name}=${docker_image}@${IMAGE_HASH} --record
          '''
        }
      }
    }
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