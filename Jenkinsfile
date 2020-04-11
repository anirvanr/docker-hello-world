#!groovy

def charts
def versions
def namespace
def addValues
def chosen_chart = "${params.charts}"

node {  
  sh "/usr/local/bin/helm repo update chartmuseum"
  chartname = sh (script: "/usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print \$1}}' | awk -F/ '{print \$2}'", returnStdout: true).trim()
}

pipeline {
  agent any

  parameters {
          choice(name: 'DryRun', choices:"Yes\nNo", description: "Do you whish to do a dry run?" )
          choice(name: 'charts', choices:"${chartname}", description: "Which Chart do you want to deploy?")
  }

stages {
  stage("parameterizing") {
    steps {
        script {
          if ("${params.DryRun}" == "Yes") {
          currentBuild.result = 'ABORTED'
          error('DRY RUN COMPLETED. JOB PARAMETERIZED.')
        }
      }
    }
  }
  stage("installed charts") {
    steps {
      script{
        sh "/usr/local/bin/helm ls --deployed $chosen_chart --output yaml"
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
  stage("choose version") {
    steps {
        script {
          def version_collection
          version_collection = sh (script: "/usr/local/bin/helm search --versions $chosen_chart | awk '{if (NR!=1) {print \$2}}'", returnStdout: true).trim()
          versions = input message: 'Choose version!', parameters: [choice(name: 'version', choices: "${version_collection}", description: '')]
        }   
      }
    }
  stage("list values") {
    steps {
      script{
        sh "/usr/local/bin/helm fetch chartmuseum/$chosen_chart --untar --untardir /tmp/charts --version $versions && cat /tmp/charts/$chosen_chart/$namespace-values.yaml"
        }
      }
    }
  stage("deploy") {
    steps {
      script{
        addValues = input message: 'Choose values!', parameters: [string(name: 'values', defaultValue: '', description: 'Any values to overwrite?')]
        if ( "${params.values}".isEmpty()​​ ) {
          sh """
            /usr/local/bin/helm upgrade --install $chosen_chart-$namespace --namespace $namespace chartmuseum/$chosen_chart --dry-run
          """
        } else {
          sh """
            /usr/local/bin/helm upgrade --install $chosen_chart-$namespace --set-string $addValues --namespace $namespace chartmuseum/$chosen_chart --dry-run
          """
          }
        }
      }
    } 
  stage("status") {
    steps {
      script{
        sh "/usr/local/bin/helm ls --deployed $chosen_chart --namespace $namespace --output yaml"
        }
      }
    }           
  }
}