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
              sh "dotnet restore sample-dotnet-app"
              sh "dotnet test ./unit-testing-using-dotnet-test"
            }
        }
    }
}