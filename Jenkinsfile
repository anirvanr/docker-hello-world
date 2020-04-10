#!groovy

def charts
def versions

node {
  charts = sh (script: "/usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print \$1}}' | awk -F \/ '{print \$2}'", returnStdout: true).trim()
}

pipeline {
  agent any

    parameters {
            choice(name: 'Invoke_Parameters', choices:"Yes\nNo", description: "Do you whish to do a dry run to grab parameters?" )
            choice(name: 'Choose Chart!', choices:"${charts}", description: "")
    }

stages {
    stage("parameterizing") {
        steps {
            script {
                if ("${params.Invoke_Parameters}" == "Yes") {
                    currentBuild.result = 'ABORTED'
                    error('DRY RUN COMPLETED. JOB PARAMETERIZED.')
                }
            }
        }
    }
  stage("choose version") {
    steps {
        script {
          def version_collection
          def chosen_chart = "${params.charts}"
          version_collection = sh (script: "/usr/local/bin/helm search $chosen_chart | awk '{if (NR!=1) {print \$2}}'", returnStdout: true).trim()
          versions = input message: 'Choose version!', ok: 'SET', parameters: [choice(name: 'Chart Version to deploy', choices: "${version_collection}", description: '')]
        }   
      }
    }              
  }
}