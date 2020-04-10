#!groovy

def deployEnv

def kubectlTest() {
    // Test that kubectl can correctly communication with the Kubernetes API
    echo "running kubectl test"
    sh "/usr/local/bin/kubectl get nodes"
}

def helmLint(String chart_dir, String deployEnv) {
    // lint helm chart
    sh "/usr/local/bin/helm lint ${chart_dir} -f ${chart_dir}/${deployEnv}-values.yaml"
}

def helmPush(String chart_dir) {
    sh """
      /usr/local/bin/helm init --client-only
      /usr/local/bin/helm repo add chartmuseum https://chartmuseum.dynacommercelab.com/techm/megafon
      /usr/local/bin/helm push --context-path=/techm/megafon ${chart_dir} chartmuseum --username ${helm_user} --password ${helm_pass}
    """
}

def helmDeploy(Map args) {
    //configure helm client and confirm tiller process is installed
    if (args.dry_run) {
        println "Running dry-run deployment"
        sh "/usr/local/bin/helm upgrade --dry-run --debug --install ${args.name}-${args.env} ${args.chart_dir} -f ${args.chart_dir}/${args.env}-values.yaml --namespace=${args.env}"
    } else {
        println "Running deployment"
        sh "/usr/local/bin/helm upgrade --recreate-pods --install ${args.name}-${args.env} ${args.chart_dir} -f ${args.chart_dir}/${args.env}-values.yaml --namespace=${args.env}"
        echo "Application ${args.name} successfully deployed in ${args.env}. Use helm status ${args.name}-${args.env} to check"
    }
}

def charts

node {
  sh """
  /usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print \$1}}' | awk -F / '{print \$2}' > commandResult
  """
  charts = readFile('commandResult').trim()
// charts = readFile('commandResult').trim()
        // charts = sh (script: "/usr/local/bin/helm search chartmuseum/ | awk '{if (NR!=1) {print $1}}' | awk -F / '{print $2}'", returnStdout: true).trim()
}

pipeline {
  agent any

  options {
    timeout(time: 15, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  environment {
    container_name = "hello-world"
    nexus_url = "dk.dynacommercelab.com"
    nexus_creds_id = "nexus-credentials"
    docker_image = "${nexus_url}/${container_name}"
    docker_tag = "1.0.3"
    // helm_repo_cred = credentials('helm-repo-creds')
  }

  parameters {
    choice(
        name: 'chartname',
        description: 'Which Chart do you want to build?',
        choices: "${charts}"
    )
    string(
        name: 'version',
        description: 'Chart Version to deploy?',
        defaultValue: '0.1.0'
    )
    string(
        name: 'values',
        description: 'Any values to overwrite?',
        defaultValue: 'key1=val1,key2=val2'
    )
  }
stages { 
  stage('Manual Deployment'){
  when { expression { BRANCH_NAME ==~ /develop/ } }
  steps{
      script {
          if (params.version) { env.addVersion = "--version ${params.version}" }
          else { env.addVersion = ' '}
          if (params.values) { env.addValues = "--set-string ${params.values}"}
          else { env.addValues = ' '  }
          switch(params.chartname) {
              case 'dev':
                  env.namespace = "--namespace development";
              break;
              default:
                  env.namespace = '--namespace default';
          }
      }      
      sh """
      /usr/local/bin/helm repo update
      /usr/local/bin/helm --install canary-${params.chartname} --atomic --wait --timeout 20s ${env.addVersion} ${env.addValues} ${env.namespace} --debug incubator/${params.chartname}
      """
      }
    }
  }
}