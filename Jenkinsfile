// Define global variable to hold dynamically loaded modules
// Modules will be loaded in 'Initialize' step
def modules = [:]

pipeline {
  options{
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  parameters {
    string(name: 'helmChartDirectory', 
           defaultValue: 'deployment/helm-k8s', 
           description: 'Relative path to Helm chart and templates')
    string(name: 'sourceRegistry', 
           defaultValue: 'mcr.microsoft.com/dotnet/sdk:6.0', 
           description: 'Registry where image will be pushed for long term storage')
    // string(name: 'sourceRegistry', defaultValue: 'quay.io/buildah/stable', description: 'Registry where image will be pushed for long term storage')
    string(name: 'targetRegistry', 
           defaultValue: 'gcr.io/cb-thunder-v2/dotnot-api', 
           description: 'Registry where image will be pushed for long term storage')
    string(name: 'prodIngressHost', 
           defaultValue: 'ameris.cb-sa.io', 
           description:'Ingress Host to set when deploying in Production environment.')

  }

  environment {
    helmChartDirectory = "${helmChartDirectory}"
    helmChartFile      = "${helmChartDirectory + '/Chart.yaml'}"
  }


  agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: mcr.microsoft.com/dotnet/sdk:6.0
    env:
      'DOTNET_CLI_HOME': '/tmp'
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'shell'
        }
    }

    stages {
      stage('Initialize') {

            agent any

            steps {

                // set build version from helm chart and current branch
                script {

                    // load modules
                    modules.helm   = load './jenkins/groovy/helm.groovy'
                    modules.common = load './jenkins/groovy/commonutils.groovy'

                    // Read Pod templates for dynamic slaves from files
                    env.buildahAgentYaml = readFile './jenkins/agents/buildah-agent.yml'
                    env.helmAgentYaml    = readFile './jenkins/agents/helm-agent.yml'
                    env.dotnetAgentYaml    = readFile './jenkins/agents/dotnet-agent.yml'

                    // Set version information to build environment
                    env.buildVersion         = modules.common.getVersionFromHelmChart(helmChartFile, releaseBranch)
                    // Set version + "git commit hash" information to environment
                    // env.buildVersionWithHash = env.buildVersion + '-' + sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                }
            }
        }
        // TODO xUnit viz in Jenkins
        stage('Unit Test') {
            steps {
              sh 'dotnet --version; ls -l /usr/bin/dotnet; which dotnet'
              sh 'ls ./sample-dotnet-app'
              sh 'dotnet build -o /tmp/dotnet/build/ unit-testing-using-dotnet-test.sln'
              sh "dotnet restore -o /tmp/dotnet/build/ sample-dotnet-app"
              sh "dotnet test -o /tmp/dotnet/build/ ./unit-testing-using-dotnet-test"
            }
        }

        // TODO Add keys  
        stage('Code Coverage Scan') {
        
            steps {
              withSonarQubeEnv(installationName:'sqA')
              sh 'dotnet sonarscanner begin /k:"ameris-bank" /d:sonar.host.url="https://sonar.newyorklifepoc.cb-demos.io"  /d:sonar.login="06fafcc8334a4128908c456afe18dbcb86eb75d3"'
              sh 'dotnet build'
              sh 'dotnet sonarscanner end /d:sonar.login="06fafcc8334a4128908c456afe18dbcb86eb75d3"'
              // sh "./mvnw clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar"

            }
        }

        stage('Deploy to Staging - example 1') {
          when { anyOf { branch releaseBranch; branch mainBranch } }
          agent {
            kubernetes {
                    yaml '''
                    apiVersion: v1
                    kind: Pod
                    spec:
                      containers:
                      - name: helm
                        image: alpine/helm
                        env:
                          DOTNET_CLI_HOME: '/tmp'
                        command:
                        - sleep
                        args:
                        - infinity
                    '''
                      defaultContainer 'helm'
                  }
              }

            steps {
              sh 'export HOME="`pwd`'
              sh 'kubectl config set-cluster development --server=$k8sCluster --insecure-skip-tls-verify'
              sh 'kubectl config set-credentials jenkins --token=$clusterAuthToken'
              sh 'kubectl config set-context helm --cluster=development --namespace=$namespace --user=jenkins'
              sh 'kubectl config use-context helm'

              sh 'helm upgrade --install --wait \
                  --namespace $namespace \
                  --set ingress.host=$ingressHost \
              
            }
        }


        stage('Deploy to Staging - example 2') {

            when { anyOf { branch releaseBranch; branch mainBranch } }

            // 'Deploy' agent pod template
            agent {
                kubernetes {
                    cloud drCloudAgents
                    label 'helm'
                    yaml helmAgentYaml
                }
            }

            container('helm') steps {

                // Deploy to K8s using helm install
                container('helm') {

                    script {

                        // by default use values for dev envrionment
                        def clusterAuthId   = devClusterAuthCredentialId
                        def namespace       = developmentNamespace
                        def ingressHost     = devIngressHost
                        def releaseName     = appName + '-dev'
                        def imagePullPolicy = 'Always'

                        // if on release branch, override them for QA environment
                        if(releaseBranch.equals(BRANCH_NAME)) {
                            clusterAuthId   = qaClusterAuthCredentialId
                            namespace       = qaNamespace
                            ingressHost     = qaIngressHost
                            releaseName     = appName + '-qa'
                            imagePullPolicy = 'IfNotPresent'
                        }

                        withCredentials([usernamePassword(credentialsId: clusterAuthId, usernameVariable: 'clusterUrl', passwordVariable: 'token')]) {
                            script {

                                // define Helm install context
                                def context              = modules.helm.newInstallContext()
                                context.tillerNS         = tillerNS
                                context.k8sCluster       = clusterUrl
                                context.clusterAuthToken = token
                                context.namespace        = namespace
                                context.appVersion       = buildVersionWithHash
                                context.ingressHost      = ingressHost
                                context.imageRepo        = imageRepo
                                context.imageTag         = buildVersion
                                context.imagePullPolicy  = imagePullPolicy
                                context.releaseName      = releaseName
                                context.chartDirectory   = helmChartDirectory

                                // run Helm install
                                modules.helm.install(context)
                            }
                        }
                    }

                }
            }
        }
    }
}