pipeline {
    agent any

    environment {
        RECIPIENT = 'wien.m.scheibelreiter@gmail.com'
    }

    stages {
        stage('Test npm ohne Docker') {
            steps {
                script {
                   def jobName = env.JOB_NAME
                    def buildNumber = env.BUILD_NUMBER
                   sh """
                      echo 'Build ${jobName} #${buildNumber}\n' > b.txt
                      ls -la
                    """
                }
            }
        }
        stage('Test npm mit Docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    def jobName = env.JOB_NAME
                    def buildNumber = env.BUILD_NUMBER

                    // Datei erstellen
                   sh """
                      npm --version
                      echo 'Build ${jobName} #${buildNumber}\n' > t.txt
                      ls -la
                    """

                    stash includes: 't.txt', name: 'npm-report'
                }
            }
        }
    }

    post {
        always {
            unstash 'npm-report'
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """<html>
                                <body>
                                    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                                        <h2>${jobName} - Build ${buildNumber}</h2>
                                        <div style="background-color:${bannerColor}; padding: 10px;">
                                            <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                                        </div>
                                        <p>Check the <a href="${env.BUILD_URL}">console output</a>.</p>
                                    </div>
                                </body>
                            </html>"""

                // Überprüfe, ob die Datei existiert
                sh "ls -l ${WORKSPACE}/t.txt"

                emailext(
                    subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: body,
                    to: RECIPIENT,
                    from: 'postmaster@sandbox09189f2badb140c494b665d48c09adca.mailgun.org',
                    replyTo: RECIPIENT,
                    mimeType: 'text/html',
                    attachmentPattern: "b.txt"
                )
            }
        }
    }
}
