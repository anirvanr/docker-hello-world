#!groovy

def charts
def version
def environment
def chart_args
def chart_name
def info(message) {
    sh "set +x ; echo '\033[0;32m===> \033[0;34m${message}\033[0;32m <===\033[0m'"
}

pipeline {
  agent any

  options {
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }
  
  parameters {
    choice(name: 'Start ', choices:"No\nYes", description: "What do you want to do?" )
  }

stages {
  stage("Initializing") {
    steps {
        script {
          if ("${params['Start ']}" == "No") {
          currentBuild.result = 'ABORTED'
          error('Aborting the build.')
        }
      }
    }
  }
  stage("Update repo") {
    steps {
      info ("Updating helm client repository information")
      script{
        sh """
        set +x
        echo "\033[0;35m\$(/usr/local/bin/helm repo add chartmuseum https://chartmuseum.dynacommercelab.com/techm/megafon \
        && /usr/local/bin/helm repo update chartmuseum)\033[0m"
        """
      }
    }
  }
  stage("Choose chart") {
    steps {
      script{
        chosen_chart = sh (script: "set +x ; /usr/local/bin/helm search chartmuseum/ | \
        awk '{if (NR!=1) {print \$1}}' | awk -F/ '{print \$2}'", returnStdout: true).trim()
        timeout(time: 1, unit: "MINUTES") {
        chart_name = input message: 'Choose chart!', parameters:
        [choice(name: 'charts', choices:"${chosen_chart}", description: "Which chart do you want to deploy?")]
        }
      }
    }
  }
  stage("View installed charts") {
    steps {
      info ("Checking the information of our deployed chart")
      script{
        sh """
        set +x
        echo "\033[0;35m\$(/usr/local/bin/helm ls --deployed $chart_name --output json | \
        jq -r .Releases[] | sed 's/{//g;s/}//g;s/"//g;s/,//g;s/^[ \t]*//g;/./,\$!d' | cat -s)\033[0m"
        """
      }
    }
  }
  stage("Choose environment") {
    steps {
      script{
        timeout(time: 1, unit: "MINUTES") {
        environment = input message: 'Choose namespace!', parameters:
        [choice(name: 'namespace', choices: "development\nproduction", description: '')]
        }
      }
    }
  }
  stage("Choose version") {
    steps {
      script {
        def version_collection
        version_collection = sh (script: "set +x ; /usr/local/bin/helm search --versions \
        $chart_name | awk '{if (NR!=1) {print \$2}}'", returnStdout: true).trim()
        echo "\033[0;35m $version_collection \033[0m"
        timeout(time: 1, unit: "MINUTES") {
        version = input message: 'Choose version!', parameters: [choice(name: 'version', choices: "${version_collection}", description: '')]
        }
      }   
    }
  }
  stage("List of values") {
    steps {
      info ("Downloading $environment-values.yaml from a repository to the local filesystem")
      script {
        env.tmp_dir = sh(script: 'set +x ; mktemp -d -t chart-XXXXX', , returnStdout: true).trim()
        sh """
        set +x
        /usr/local/bin/helm fetch chartmuseum/$chart_name --untar --untardir $tmp_dir --version $version
        """
        def status_code = sh(script: "set +x ; cat $tmp_dir/$chart_name/$environment-values.yaml", returnStatus: true)
          if ( status_code != 0 ) {
            currentBuild.result = 'FAILED'
            error('The script failed')
        }
      }
    }
  }
  stage("Deploy chart") {
    steps {
      script{
        timeout(time: 10, unit: "MINUTES") {
        chart_args = input message: 'Choose values!', parameters: 
        [string(name: 'values', defaultValue: 'none', description: 'Any values to overwrite?')]
        }
        info ("Deploying helm chart")
        sh """
        set +x
        if [[ $chart_args = "none" ]]
        then
          echo "\033[0;35m\$(/usr/local/bin/helm upgrade --install $chart_name-$environment --namespace \
          $environment -f $tmp_dir/$chart_name/$environment-values.yaml chartmuseum/$chart_name --dry-run)\033[0m"
        else
          echo "\033[0;35m\$(/usr/local/bin/helm upgrade --install $chart_name-$environment --set $chart_args \
          --namespace $environment -f $tmp_dir/$chart_name/$environment-values.yaml chartmuseum/$chart_name --dry-run)\033[0m"
        fi
        rm -rf $tmp_dir
        """
      }
    }
  }
  stage("Get status") {
    steps {
      info ("Checking status of helm chart")
      script{
        sh """
        set +x
        echo "\033[0;35m\$(/usr/local/bin/helm ls --deployed $chart_name --namespace $environment --output json | \
        jq -r .Releases[] | sed 's/{//g;s/}//g;s/"//g;s/,//g;s/^[ \t]*//g;/./,\$!d' | cat -s)\033[0m"
        """
        }
      }
    }           
  }
}