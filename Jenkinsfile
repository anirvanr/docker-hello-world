#!groovy

// Define variables (based on parameters set in a Jenkins job)
def charts
def version
def environment
def chart_args
def chart_name


pipeline {
  agent any

  options {
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }
  
  parameters {
    choice(name: 'dryrun', choices:"Yes\nNo", description: "Do you whish to do a dry run?" )
  }

stages {
  stage("Parameterizing") {
    steps {
        script {
          if ("${params.dryrun}" == "Yes") {
          currentBuild.result = 'ABORTED'
          error('DRY RUN COMPLETED. JOB PARAMETERIZED.')
        }
      }
    }
  }
  stage("Update repo") {
    steps {
      script{
        sh """
        set +x
        echo "\033[0;32m===> \033[0;34mUpdating helm client repository information\033[0;32m <=== \033[0m"
        /usr/local/bin/helm repo add chartmuseum https://chartmuseum.dynacommercelab.com/techm/megafon
        /usr/local/bin/helm repo update chartmuseum
        """
      }
    }
  }
  stage("Choose chart") {
    steps {
      script{
        chosen_chart = sh (script: "/usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print \$1}}' | awk -F/ '{print \$2}'", returnStdout: true).trim()
        timeout(time: 1, unit: "MINUTES") {
        chart_name = input message: 'Choose chart!', parameters: [choice(name: 'charts', choices:"${chosen_chart}", description: "Which Chart do you want to deploy?")]
        }
      }
    }
  }
  stage("View installed charts") {
    steps {
      script{
        sh """
        set +x
        echo "\033[0;32m===> \033[0;34mChecking the information of our deployed chart\033[0;32m <=== \033[0m"
        /usr/local/bin/helm ls --deployed $chart_name --output yaml
        """
      }
    }
  }
  stage("Choose environment") {
    steps {
      script{
        timeout(time: 1, unit: "MINUTES") {
        environment = input message: 'Choose namespace!', parameters: [choice(name: 'namespace', choices: "development\nproduction", description: '')]
        }
      }
    }
  }
  stage("Choose version") {
    steps {
      script {
        def version_collection
        version_collection = sh (script: "/usr/local/bin/helm search --versions $chart_name | awk '{if (NR!=1) {print \$2}}'", returnStdout: true).trim()
        timeout(time: 1, unit: "MINUTES") {
        version = input message: 'Choose version!', parameters: [choice(name: 'version', choices: "${version_collection}", description: '')]
        }
      }   
    }
  }
  stage("List of values") {
    steps {
      script{
        env.tmp_dir = sh(script: 'mktemp -d -t chart-XXXXXXXXXX', , returnStdout: true).trim()
        sh """
        echo $tmp_dir
        echo "\033[0;32m===> \033[0;34mDownloading $environment-values.yaml from a repository to the local filesystem\033[0;32m <=== \033[0m"
        /usr/local/bin/helm fetch chartmuseum/$chart_name --untar --untardir $tmp_dir --version $version && cat $tmp_dir/$chart_name/$environment-values.yaml
        """
      }
    }
  }
  stage("Deploy chart") {
    steps {
      script{
        timeout(time: 10, unit: "MINUTES") {
        chart_args = input message: 'Choose values!', parameters: [string(name: 'values', defaultValue: 'none', description: 'Any values to overwrite?')]
        }
        sh """
        echo "\033[0;32m===> \033[0;34mDeploying helm chart\033[0;32m <=== \033[0m"
        if [[ $chart_args = "none" ]]
        then
          /usr/local/bin/helm upgrade --install $chart_name-$environment --namespace $environment -f $tmp_dir/$chart_name/$environment-values.yaml chartmuseum/$chart_name --dry-run
        else
          /usr/local/bin/helm upgrade --install $chart_name-$environment --set-string $chart_args --namespace $environment -f $tmp_dir/$chart_name/$environment-values.yaml chartmuseum/$chart_name --dry-run
        fi
        rm -rvf $tmp_dir
        """
      }
    }
  }
  stage("Get status") {
    steps {
      script{
        sh """
        set +x
        echo "\033[0;32m===> \033[0;34mChecking status of helm chart\033[0;32m <=== \033[0m"
        /usr/local/bin/helm ls --deployed $chart_name --namespace $environment --output yaml
        """
        }
      }
    }           
  }
}