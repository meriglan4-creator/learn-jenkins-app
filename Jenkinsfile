pipeline {

    agent any

    environment {
        NETLIFY_SITE_ID = '4844b1b2-c017-4c6e-b224-4bf1274c67c7'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }


    stages {

        stage('Build') {
            agent {
                docker {
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
                    ls -la build
                '''
            }
        }


        stage('Tests') {

            parallel {

                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            CI=true npm test -- --reporters=default --reporters=jest-junit
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
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            npx serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: true,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright local',
                                reportTitles: ''
                            ])
                        }
                    }
                }

            }
        }


        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify deploy \
                        --dir=build \
                        --prod \
                        --site=$NETLIFY_SITE_ID \
                        --auth=$NETLIFY_AUTH_TOKEN \
                        --no-build
                '''
            }
        }


        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://tiny-monstera-bb890a.netlify.app'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright Production E2E',
                        reportTitles: ''
                    ])
                }
            }
        }

    }

}
