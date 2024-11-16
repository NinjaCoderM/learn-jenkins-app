pipeline {
    agent any

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
                    ls -la
                '''
            }
        }
        stage('test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   echo 'Test stage'
                   ls -l build/index.html
                   npm test
                '''
            }
        }
        stage('E2E') {
                    agent {
                        docker {
                            //Quelle für E2E playright
                            //https://playwright.dev/
                            //Quelle image docker
                            //https://playwright.dev/docs/docker
                            image 'mcr.microsoft.com/playwright:v1.48.1-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                           echo 'E2E stage'
                           npm install -g serve
                           serve -s build
                           npx playwright test
                        '''
                    }
                }
    }
    post{
        always{
           junit 'test-results/junit.xml'
        }
    }
}