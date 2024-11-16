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
        stage('run tst'){
            parallel{
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
                            // Quelle für E2E playright
                            // https://playwright.dev/
                            // Quelle image docker
                            // https://playwright.dev/docs/docker
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            // args '-u root:root'
                        }
                    }
                    // npm install -g serve //-> get nur mit root
                    // serve -s build //-> nur bei npm install -g serve
                    steps {
                        sh '''
                           echo 'E2E stage'
                           echo 'innerhalb des Projekts unter node_modules/.bin installieren mit *npm install serve*'
                           npm install serve
                           echo 'geht ohne root durch weglassen von -g in npm install serve mit *node_modules/.bin/serve -s build &*'
                           echo '*&* is used to run server in background !!!'
                           node_modules/.bin/serve -s build &
                           echo 'time for server to start *sleep 10*'
                           sleep 10
                           echo 'use *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                           echo 'nicht empfohlen in Produktion -> *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                           echo 'sic *;)"*'
                           npx playwright test --reporter=html
                        '''
                    }
                }
            }
        }
    }

    post{
        always{
           junit 'jest-results/junit.xml'
           sh "echo 'Install Plugin html Publisher in Jenkins'"
           sh "echo 'Use Pipeline Syntax at the End of the Pipeline in jenkins! Choose publishHTML: Publish HTML Report and add directory playwright-report -> The generated Line is seen in next line! '"
           publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}