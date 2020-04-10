#!groovy

def charts
def versions

node {
  charts = sh (script: "/usr/local/bin/helm search chartmuseum | awk '{if (NR!=1) {print \$1}}'", returnStdout: true).trim()
}

pipeline {
  agent any

    parameters {
            choice(name: 'Choose Chart!', choices:"${charts}", description: "")
    }

stages {
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