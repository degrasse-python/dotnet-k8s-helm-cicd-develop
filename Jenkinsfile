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
    tools{
      jdk 'openjdk-9'
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
              //sh 'chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/bin/*'
              //sh 'chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/lib/*.jar'
              sh 'chmod 775 /root/.dotnet/tools/.store/dotnet-sonarscanner/5.8.0/dotnet-sonarscanner/5.8.0/tools/net5.0/any/sonar-scanner-4.7.0.2747/bin/sonar-scanner'
              sh 'chmod 775 /root/.dotnet/tools/.store/dotnet-sonarscanner/5.8.0/dotnet-sonarscanner/5.8.0/tools/net5.0/any/sonar-scanner-4.7.0.2747/lib/sonar-scanner-cli-4.7.0.2747.jar'
              sh 'dotnet restore ./unit-testing-using-dotnet-test/PrimeService.Tests/'
              sh 'dotnet test ./unit-testing-using-dotnet-test/PrimeService.Tests/ --logger:"xunit;LogFilePath=/var/log/test_result.xml"'
              sh 'cat /var/log/test_result.xml'
              xunit (tools: [ BoostTest(pattern: '/var/log/*.xml') ], skipPublishingChecks: false)
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