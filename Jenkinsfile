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
  - name: shell
    image: mcr.microsoft.com/dotnet/sdk:6.0
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'shell'
        }
    }
    environment {
        DOTNET_CLI_HOME = "/tmp"
    }
    // TODO xUnit viz in Jenkins
    stages {
        stage('Unit Test') {
            steps {
              sh 'dotnet --version; ls -l /usr/bin/dotnet; which dotnet'
              sh 'dotnet build -o /tmp/dotnet/build/ unit-testing-using-dotnet-test.sln'
              sh "dotnet restore -o /tmp/dotnet/build/ sample-dotnet-app"
              sh "dotnet test -o /tmp/dotnet/build/ ./unit-testing-using-dotnet-test"
            }
        }

    }
}