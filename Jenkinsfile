pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:10.0' 
            args '-e HOME=/tmp -e DOTNET_CLI_HOME=/tmp -e NUGET_PACKAGES=/tmp/.nuget/packages'
        }
    }

    stages {
        stage('Restore') {
            steps {
                sh 'dotnet restore dotnet-demo-app/TodoApp.Tests'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    dotnet test dotnet-demo-app/TodoApp.Tests \
                      --no-restore \
                      --logger "trx;LogFileName=results.trx" \
                      --results-directory TestResults
                '''
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResultsFileRegex: 'TestResults/*.trx'
        }
    }
}
