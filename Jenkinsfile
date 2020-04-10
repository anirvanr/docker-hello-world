#!groovy

def charts
def versions

node {
  charts = sh (script: "/usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print \$1}}'", returnStdout: true).trim()
}

pipeline {
  agent any

    parameters {
            choice(name: 'Invoke_Parameters', choices:"Yes\nNo", description: "Do you whish to do a dry run to grab parameters?" )
            choice(name: 'charts', choices:"${charts}", description: "")
    }

stages {

  stage("choose version") {
    steps {
        script {
          def version_collection
          def chosen_chart = "${params.charts}"
          version_collection = sh (script: "/usr/local/bin/helm search $chosen_chart | awk '{if (NR!=1) {print \$2}}'", returnStdout: true).trim()
          versions = input message: 'Choose testload version!', ok: 'SET', parameters: [choice(name: 'Chart Version to deploy', choices: "${version_collection}", description: '')]
      }
    }
  }              
  // stage('Manual Deployment'){
  // when { expression { BRANCH_NAME ==~ /develop/ } }
  // steps{
  //     script {
  //         if (params.version) { env.addVersion = "--version ${params.version}" }
  //         else { env.addVersion = ' '}
  //         if (params.values) { env.addValues = "--set-string ${params.values}"}
  //         else { env.addValues = ' '  }
  //         switch(params.chartname) {
  //             case 'dev':
  //                 env.namespace = "--namespace development";
  //             break;
  //             default:
  //                 env.namespace = '--namespace default';
  //         }
  //     }      
  //     sh """
  //     /usr/local/bin/helm repo update
  //     /usr/local/bin/helm --install canary-${params.chartname} --atomic --wait --timeout 20s ${env.addVersion} ${env.addValues} ${env.namespace} --debug incubator/${params.chartname}
  //     """
  //     }
  //   }
  }
}