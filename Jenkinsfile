pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'cb8e431a-50ce-42f6-a263-686d2bfa244f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages { 
        stage('Build') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                   ls -la
                   node --version
                   npm --version
                   npm ci
                   npm run build
                   ls -la 
                '''
            }
        }
        stage ('Tests'){
             parallel {
                    stage('Unit tests') {
                        agent {
                            docker{
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }
                        steps {
                            sh '''
                                #test -f build/index.html
                                npm test
                            '''
                        }
                        post {
                            always {
                               junit 'jest-results/junit.xml'
                            }
                        }
                    }
                    
                    stage('E2E') {
                        agent {
                            docker{
                                image 'my-playwright'
                                reuseNode true
                            }
                        }

                        steps {
                            sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                            '''
                        }
                        post{
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                    }
             }
        }

        stage('Deploy Staging') {
                        agent {
                            docker{
                                image 'my-playwright'
                                reuseNode true
                            }
                        }

                        environment {
                            CI_ENVIRONMENT_URL = 'staging url to be set'
                        }

                        steps {
                            sh '''
                                netlify --version
                                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                                netlify status
                                netlify deploy --dir=build --json > deploy-output.json
                                CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                                npx playwright test --reporter=html
                            '''
                        }
                        post{
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                    } 

        stage('Deploy Prod') {
                        agent {
                            docker{
                                image 'my-playwright'
                                reuseNode true
                            }
                        }

                        environment {
                            CI_ENVIRONMENT_URL = 'https://velvety-bienenstitch-c30023.netlify.app'
                        }

                        steps {
                            sh '''
                                node --version
                                netlify --version
                                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                                netlify status
                                netlify deploy --dir=build --prod
                                npx playwright test --reporter=html
                            '''
                        }
                        post{
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                    }
    }
}
