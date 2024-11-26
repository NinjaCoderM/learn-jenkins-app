pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c6e7d834-e196-410f-a226-dc7afb62bc8a'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        stage('Docker'){
            steps{
                sh 'docker build -t my-playwright .'
            }
        }
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

        stage('AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli:latest'
                    args '--entrypoint=""'
                }
            }
            environment{
                AWS_S3_BUCKET = 'learn-jenkins-202411252156'
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 ls
                        echo "Hello S3!" > index.html
                        echo "replaced ... aws s3 cp index.html s3://$AWS_S3_BUCKET/index.html"
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }

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
                stage('E2E local') {
                    agent {
                        docker {
                            // Quelle für E2E playright!
                            // https://playwright.dev/
                            // Quelle image docker
                            // https://playwright.dev/docs/docker
                            //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            image 'my-playwright'
                            reuseNode true
                            // args '-u root:root'
                        }
                    }
                    // npm install -g serve //-> get nur mit root
                    // serve -s build //-> nur bei npm install -g serve
                    steps {
                        sh '''
                           echo 'E2E stage local'
                           echo 'innerhalb des Projekts unter node_modules/.bin installieren mit *npm install serve*'
                           echo 'im Dockerfile mit -g *npm install serve*'
                           echo 'geht ohne root durch weglassen von -g in npm install serve mit *node_modules/.bin/serve -s build &*'
                           echo '*&* is used to run server in background !!!'
                           echo 'nicht mehr weil -g  *node_modules/.bin/*'
                           serve -s build &
                           echo 'time for server to start *sleep 10*'
                           sleep 10
                           echo 'use *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                           echo 'nicht empfohlen in Produktion -> *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                           echo 'sic *;)"*'
                           npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

            }
        }

         stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   echo 'npm install netlify-cli -g geht nicht wegen Berechtigung / keine root Rechte'
                   npm install netlify-cli
                   echo 'Aufruf local weil -g fehlt unter node_modules/.bin/netlify --version'
                   node_modules/.bin/netlify --version
                   echo "Start Deployment to staging environment... NETLIVY_SITE_ID must be set in environment: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   echo 'netlify deploy ohne prod *--dir=build prod* zu *--dir=build*--> staging environment'
                   node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                   echo 'Install jq mit node (weil einfacher) normalerweise *npm install netlify-cli node-jq* in erster Zeile'
                   npm install node-jq
                   echo 'CI_ENVIRONMENT_URL ist nur im sh Block sichtbar! Kein Abstand bei = CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json) '
                   echo 'Deploy staging könnte in Staging E2E dazugefügt werden! Bleibt aber wegen der Variable als Demo'
                   echo 'environment Block mit CI_ENVIRONMENT_URL muss bleiben, sonst wird er von playwright nicht erkannt'
                   CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                '''
                script {
                    env.netlify_response_url_staging = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true).trim()
                }
                echo "netlify_response_url_staging: ${env.netlify_response_url_staging}"
            }
        }

        stage('Staging E2E') {
                    agent {
                        docker {
                            // Quelle für E2E playright!
                            // https://playwright.dev/
                            // Quelle image docker
                            // https://playwright.dev/docs/docker
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            // args '-u root:root'
                        }
                    }
                    environment{
                        CI_ENVIRONMENT_URL = "${env.netlify_response_url_staging}"
                    }

                    steps {
                        sh '''
                           echo 'Prod E2E stage'
                           echo 'use *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                           echo 'nicht empfohlen in Produktion -> *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                           echo 'sic *;)"*'
                           npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

        /*
        stage('Approve Deployment'){
            steps{
                timeout(3) {
                    input message: 'Bitte um Bestätigung des Deployments - Anwendung jenkinsFromGit', ok: 'Yes, proceed with deploy'
                }
            }
        }
        */

        stage('Deploy Prod with E2E') {
            agent {
                docker {
                    // Quelle für E2E playright!
                    // https://playwright.dev/
                    // Quelle image docker
                    // https://playwright.dev/docs/docker
                    // image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    // eigenes Image erstellt stage Docker über Dockerfile
                    image 'my-playwright'

                    reuseNode true
                    // args '-u root:root'
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://zippy-lollipop-74598d.netlify.app'
            }

            steps {
                sh '''
                   node --version
                   echo 'npm install netlify-cli -g geht nicht wegen Berechtigung / keine root Rechte'
                   echo 'bereits von stage Docker über Dockerfile ... npm install netlify-cli'
                   echo 'Aufruf local weil -g fehlt unter node_modules/.bin/netlify --version'
                   echo 'durch Dockerfile kann global installiert werden node_modules/.bin/netlify --version'
                   netlify --version
                   echo "Start Deployment ... NETLIVY_SITE_ID must be set in environment: $NETLIFY_SITE_ID"
                   echo 'Änderung auf global node_modules/.bin/netlify status'
                   netlify status
                   echo 'Änderung auf global node_modules/.bin/netlify deploy --dir=build --prod'
                   netlify deploy --dir=build --prod
                   echo 'Prod E2E stage'
                   echo 'use *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                   echo 'nicht empfohlen in Produktion -> *System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;)"* in Jenkins verwalten / Skript Console '
                   echo 'sic *;)"*'
                   npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }

    post{
        always{
           junit 'jest-results/junit.xml'
           sh "echo 'Install Plugin html Publisher in Jenkins'"
           sh "echo 'Use Pipeline Syntax at the End of the Pipeline in jenkins! Choose publishHTML: Publish HTML Report and add directory playwright-report -> The generated Line is seen in next line! '"
        }
    }
}