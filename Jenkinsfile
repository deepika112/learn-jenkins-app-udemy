pipeline {
    agent any

    stages {
        stage('Build') {
            agent{
                Docker{
                    image 'node:18-alpine'
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm build
                    ls -la
                '''
            }
        }
    }
}
