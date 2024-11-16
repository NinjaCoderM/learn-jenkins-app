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
                            // args '-u root:root'
                        }
                    }
                    steps {
                        sh '''
                           echo 'E2E stage'
                           //npm install -g serve //-> get nur mit root
                           npm install serve // innerhalb des Projekts unter node_modules/.bin installieren
                           //serve -s build //-> nur bei npm install -g serve
                           node_modules/.bin/serve -s build //-> geht ohne root durch weglassen von -g in npm install serve
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