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
    securityContext:
          allowPrivilegeEscalation: false
          runAsUser: 0
'''
            defaultContainer 'shell'
        }
    }
    environment {
        DOTNET_CLI_HOME = "/tmp"
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
              withSonarQubeEnv(installationName:'sqA')
              sh 'dotnet sonarscanner begin /k:"ameris-bank" /d:sonar.host.url="https://sonar.newyorklifepoc.cb-demos.io"  /d:sonar.login="06fafcc8334a4128908c456afe18dbcb86eb75d3"'
              sh 'dotnet build'
              sh 'dotnet sonarscanner end /d:sonar.login="06fafcc8334a4128908c456afe18dbcb86eb75d3"'
              // sh "./mvnw clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar"

            }
        }

    }
}