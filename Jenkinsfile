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

                    steps {

                        sh '''
                            docker run --rm \
                            -v "$PWD:/app" \
                            -w /app \
                            node:18-alpine \
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

                    steps {

                        sh '''
                            docker run --rm \
                            -v "$PWD:/app" \
                            -w /app \
                            mcr.microsoft.com/playwright:v1.39.0-jammy \
                            sh -c "
                                npm install serve &&
                                serve -s build &
                                sleep 10 &&
                                npx playwright test --reporter=html
                            "
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


            steps {


                sh '''

                    docker run --rm \
                    -v "$PWD:/app" \
                    -w /app \
                    -e NETLIFY_AUTH_TOKEN="$NETLIFY_AUTH_TOKEN" \
                    node:18-alpine \
                    sh -c "

                        npm install netlify-cli &&

                        node_modules/.bin/netlify deploy \
                        --dir=build \
                        --prod \
                        --site=$NETLIFY_SITE_ID \
                        --auth=$NETLIFY_AUTH_TOKEN \
                        --no-build

                    "

                '''
            }
        }





        stage('Prod E2E') {


            environment {

                CI_ENVIRONMENT_URL = 'https://tiny-monstera-bb890a.netlify.app'

            }



            steps {


                sh '''

                    docker run --rm \
                    -v "$PWD:/app" \
                    -w /app \
                    -e CI_ENVIRONMENT_URL="$CI_ENVIRONMENT_URL" \
                    mcr.microsoft.com/playwright:v1.39.0-jammy \
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
