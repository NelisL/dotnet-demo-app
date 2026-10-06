pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                catchError {
                    sh 'docker stop todoapp'
                    sh 'docker rm todoapp'
                    sh 'docker stop todoappdb'
                    sh 'docker rm todoappdb'
                }
            }
        }
        stage('Test') {
            steps {
                build job: 'test-dotnet-demo-app'
            }
        }
        stage('Build') {
            steps {
                build job: 'dotnet-demo-app'
            }
        }
    }
}
