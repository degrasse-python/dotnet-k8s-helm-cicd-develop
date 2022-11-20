pipeline {
  options{
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dotnet-sdk
    image: docker.io/adsaunde/dotnetsdk-sonarqube:alpine
    command:
    - sleep
    args:
    - infinity
    securityContext:
          allowPrivilegeEscalation: false
          runAsUser: 0
'''
            defaultContainer 'dotnet-sdk'
        }
    }
    environment {
        DOTNET_CLI_HOME = "/tmp"
        PATH="$PATH:/tmp/.dotnet/tools"
    }
  
    stages {
        /*

         *
         * Unit Test and Code Scan
         * Only executes on main and release branch builds. 
         *
        
        */
        // TODO xUnit viz in Jenkins
        stage('Unit Test') {
            steps {
              sh 'dotnet --version; ls -l /usr/bin/dotnet; which dotnet'
              sh 'pwd'
              // sh 'dotnet restore -o /tmp/dotnet/build/ ./unit-testing-using-dotnet-test/PrimeService.Tests/'
              // sh "dotnet restore -o /tmp/dotnet/build/ sample-dotnet-app"
              sh 'dotnet restore ./unit-testing-using-dotnet-test/PrimeService.Tests/'
              sh "dotnet test ./unit-testing-using-dotnet-test/PrimeService.Tests/"
            }
        }

        // TODO Add keys  
        stage('Code Coverage Scan') {
        
            steps {
              withSonarQubeEnv(installationName:'sqA'){
              sh 'dotnet tool install --global dotnet-sonarscanner'
              sh 'dotnet sonarscanner begin /k:"ameris-bank" /d:sonar.host.url="https://sonar.newyorklifepoc.cb-demos.io"  /d:sonar.login="06fafcc8334a4128908c456afe18dbcb86eb75d3"'
              sh 'dotnet build ./unit-testing-using-dotnet-test/PrimeService'
              sh 'dotnet sonarscanner end /d:sonar.login="06fafcc8334a4128908c456afe18dbcb86eb75d3"'
              //sh "dotnet build ./unit-testing-using-dotnet-test/PrimeService/ sonar:sonar"
              }
            }
        }

    }
}