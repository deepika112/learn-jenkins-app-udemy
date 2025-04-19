pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='8e9bf951-a6d5-47d4-8f42-42eaf04fc822'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
    }
    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('All Tests'){
            parallel{
                stage ('Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            echo "Test stage"
                            test -f build/index.html
                            npm test 
                        '''
                    }
                }

                stage('E2E Test'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }
        }
        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli
                   node_modules/.bin/netlify --version
                   echo "Deploying the site"
                   node_modules/.bin/netlify status
                '''
            }
        }
    }

    post{
        always{
            junit "jest-results/junit.xml"
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlaywrightHTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
