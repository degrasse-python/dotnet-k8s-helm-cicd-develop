pipeline {
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
    - name: 'DOTNET_CLI_HOME'
      value: '/tmp'
    command:
    - sleep
    args:
    - infinity
'''
            defaultContainer 'shell'
        }
    }
    stages {
        stage('Main') {
            steps {
              sh 'dotnet --version; ls -l /usr/bin/dotnet; which dotnet'
              sh 'ls ./sample-dotnet-app'
              sh 'dotnet build -o /tmp/dotnet/build/ unit-testing-using-dotnet-test.sln'
              sh "dotnet restore -o /tmp/dotnet/build/ sample-dotnet-app"
              sh "dotnet test -o /tmp/dotnet/build/ ./unit-testing-using-dotnet-test"
            }
        }
    }
}