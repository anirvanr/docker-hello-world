#!groovy

def charts
def versions

node {  
  sh "/usr/local/bin/helm repo update chartmuseum"
  chartname = sh (script: "/usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print \$1}}'", returnStdout: true).trim()
}

pipeline {
  agent any

  parameters {
          choice(name: 'dry_run', choices:"Yes\nNo", description: "Do you whish to do a dry?" )
          choice(name: 'charts', choices:"${chartname}", description: "Which Chart do you want to deploy?")
  }

stages {
  stage("parameterizing") {
    steps {
        script {
          if ("${params.dry_run}" == "Yes") {
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
          version_collection = sh (script: "/usr/local/bin/helm search --versions $chosen_chart | awk '{if (NR!=1) {print \$2}}'", returnStdout: true).trim()
          versions = input message: 'Choose version!', parameters: [choice(name: 'version', choices: "${version_collection}", description: '')]
        }   
      }
    }
  stage("choose env") {
    steps {
      script{
        namespace = input message: 'Choose namespace!', parameters: [choice(name: 'namespace', choices: "development\nproduction", description: '')]
        }
      }
    }
  stage("view values") {
    steps {
      script{
        def chosen_chart = "${params.charts}"
        sh "/usr/local/bin/helm fetch $chosen_chart --untar --untardir /tmp/charts --version $versions && cat /tmp/charts/hello-world/"${params.namespace}"-values.yaml"
        }
      }
    }            
  }
}