pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                build job: '/test-dotnet-demo-app'
            }
        }
        stage('Build') {
            steps {
                build job: '/dotnet-demo-app'
            }
        }
    }
}
