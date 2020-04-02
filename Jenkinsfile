#!groovy

  def kubectlTest() {
    // Test that kubectl can correctly communication with the Kubernetes API
    echo "running kubectl test"
    sh "kubectl get nodes"

}

pipeline {
  agent any

  environment {
    NAME = "hello-world"
    NEXUS_URL = "https://dk.dynacommercelab.com"
    NEXUS_CREDENTIAL_ID = "nexus-credentials"
    DOCKER_IMAGE = "dk.dynacommercelab.com/${NAME}"
    TAG = "1.0.3"
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
          withDockerRegistry([ credentialsId: "${NEXUS_CREDENTIAL_ID}", url: "${NEXUS_URL}" ]){
          sh '''
          if [[ ${BRANCH_NAME} =~ master ]]
          then
            docker build -f "Dockerfile" -t ${DOCKER_IMAGE}:${TAG} .
            docker push ${DOCKER_IMAGE}:${TAG} || { >&2 echo "Failed to push tag '${TAG}' image ${DOCKER_IMAGE}"; exit 1; }
          elif [[ ${BRANCH_NAME} =~ develop ]]
          then
            docker build -f "Dockerfile" -t ${DOCKER_IMAGE}:latest .
            docker push ${DOCKER_IMAGE} || { >&2 echo "Failed to push tag 'latest' image ${DOCKER_IMAGE}"; exit 1; }
          else
            echo 'Do that only on master or develop branch'
          fi
          '''
        }
      }
    }

    stage('Deploy development') {
        when {
          expression { BRANCH_NAME ==~ /develop/ }
        }
        steps {
          withDockerRegistry([ credentialsId: "${NEXUS_CREDENTIAL_ID}", url: "${NEXUS_URL}" ]){
          sh '''
          IMAGE_HASH="$(docker pull ${DOCKER_IMAGE} | grep 'Digest: ' | sed 's/Digest: //')"
          /usr/local/bin/kubectl --namespace=development set image deployment/${NAME} ${NAME}=${DOCKER_IMAGE}@${IMAGE_HASH} --record
          '''
        }
      }
    }
    stage('Deploy production') {
        when {
          expression { BRANCH_NAME ==~ /master/ }
        }
        steps {
          withDockerRegistry([ credentialsId: "${NEXUS_CREDENTIAL_ID}", url: "${NEXUS_URL}" ]){
          sh '''
          echo ${kubectlTest}
          IMAGE_HASH="$(docker pull $DOCKER_IMAGE:${TAG} | grep 'Digest: ' | sed 's/Digest: //')"
          /usr/local/bin/kubectl --namespace=production set image deployment/${NAME} ${NAME}=${DOCKER_IMAGE}@${IMAGE_HASH} --record
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
    //       NEXUS_URL_MF = "https://dkmf.dynacommercelab.com"
    //     }
    //     steps {
    //       script {
    //           timeout(time: 1, unit: 'MINUTES') {
    //           env.IMAGE_PUSH = input message: 'User input required', ok: 'Continue',
    //           parameters: [choice(name: 'Upload docker image', choices: 'yes\nno', description: '')]
    //           }
    //       withDockerRegistry([ credentialsId: "${NEXUS_CREDENTIAL_ID}", url: "${NEXUS_URL_MF}" ]){
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