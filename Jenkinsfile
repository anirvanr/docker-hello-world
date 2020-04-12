#!groovy

// Define variables (based on parameters set in a Jenkins job)
def charts
def version
def environment
def deploy_args
def deployment_name = params.charts

node {  
  sh """
  set +x
  echo "\033[0;34m Updating helm client repository information \033[0m"
  /usr/local/bin/helm repo add chartmuseum https://chartmuseum.dynacommercelab.com/techm/megafon
  /usr/local/bin/helm repo update chartmuseum
  """
  chartname = sh (script: "/usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print \$1}}' | awk -F/ '{print \$2}'", returnStdout: true).trim()
}

pipeline {
  agent any

  options {
    timeout(time: 60, unit: 'MINUTES')
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  
def userInput = input( id: 'userInput', message: 'Choose chart!', parameters [
          [choice(name: 'dryrun', choices:"Yes\nNo", description: "Do you whish to do a dry run?" )]
          [choice(name: 'charts', choices:"${chartname}", description: "Which Chart do you want to deploy?")]
])

stages {
  stage("parameterizing") {
    steps {
        script {
          if ("${params.dryrun}" == "Yes") {
          currentBuild.result = 'ABORTED'
          error('DRY RUN COMPLETED. JOB PARAMETERIZED.')
        }
      }
    }
  }
  stage("installed charts") {
    steps {
      script{
        sh """
        set +x
        echo "\033[0;34m Check the information of our deployed chart \033[0m"
        /usr/local/bin/helm ls --deployed $deployment_name --output yaml
        """
        }
      }
    }
  stage("choose env") {
    steps {
      script{
        environment = input message: 'Choose namespace!', parameters: [choice(name: 'namespace', choices: "development\nproduction", description: '')]
        }
      }
    }
  stage("choose version") {
    steps {
        script {
          def version_collection
          version_collection = sh (script: "/usr/local/bin/helm search --versions $deployment_name | awk '{if (NR!=1) {print \$2}}'", returnStdout: true).trim()
          version = input message: 'Choose version!', parameters: [choice(name: 'version', choices: "${version_collection}", description: '')]
        }   
      }
    }
  stage("list values") {
    steps {
      script{
        sh """
        set +x
        echo "\033[0;34m Fetch a reference $environment-values.yaml \033[0m"
        /usr/local/bin/helm fetch chartmuseum/$deployment_name --untar --untardir /tmp/charts --version $version && cat /tmp/charts/$deployment_name/$environment-values.yaml
        """
        }
      }
    }
  stage("deploy") {
    steps {
      script{
      deploy_args = input message: 'Choose values!', parameters: [string(name: 'values', defaultValue: 'none', description: 'Any values to overwrite?')]
      sh """
      set +x
      echo "\033[0;34m Installing the $deployment_name Helm Chart \033[0m"
      if [[ $deploy_args = "none" ]]
        then
            /usr/local/bin/helm upgrade --install $deployment_name-$environment --namespace $environment chartmuseum/$deployment_name --dry-run
        else
          /usr/local/bin/helm upgrade --install $deployment_name-$environment --set-string $deploy_args --namespace $environment chartmuseum/$deployment_name --dry-run
      fi
      """
      }
    }
  }
  stage("status") {
    steps {
      script{
        sh """
        set +x
        echo "\033[0;34m check status of deployment \033[0m"
        /usr/local/bin/helm ls --deployed $deployment_name --namespace $environment --output yaml
        """
        }
      }
    }           
  }
}