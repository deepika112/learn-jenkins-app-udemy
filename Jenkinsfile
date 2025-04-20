pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='8e9bf951-a6d5-47d4-8f42-42eaf04fc822'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
        REACT_APP_VERSION="1.0.$BUILD_ID"
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

        stage('DOCKER'){
            steps{
                sh 'docker build -t playwright-image .'
            }
        }
        stage('All Tests'){
            parallel{
                stage ('Test'){
                    agent{
                        docker{
                            image 'playwright-image'
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
                    post{
                        always{
                            junit "jest-results/junit.xml"
                        }
                    }
                }

                stage('E2E Test'){
                    agent{
                        docker{
                            image 'playwright-image'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local Test Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   netlify --version
                   echo "Deploying the site"
                   netlify status
                   netlify deploy --dir=build --json > deploy-staging.txt
                   # --prod removed the flag to create a staging environment 
                   # for netlify which is the default if not specified
                '''
                script{
                    env.Deploy_URL=sh(script: "node-jq -r '.deploy_url' deploy-staging.txt", returnStdout: true)
                }
            }
        }

        stage('Staging E2E Test'){
            agent{
                docker{
                    image 'playwright-image'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL="${env.Deploy_URL}"
            }
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval'){
            steps{
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy Production') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   netlify --version
                   echo "Deploying the site"
                   netlify status
                   netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Production E2E Test'){
            agent{
                docker{
                    image 'playwright-image'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL='https://aquamarine-meringue-1026ed.netlify.app/'
            }
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Production E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
