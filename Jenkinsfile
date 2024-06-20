pipeline {
    agent any

    environment {
        RECIPIENTS = 'hujefashaik346@gmail.com' // Replace with actual email address
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    captureAndSendErrorLogs('Checkout') {
                        checkout scm
                    }
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    captureAndSendErrorLogs('Build') {
                        sh 'mvn clean compile'
                    }
                }
            }
        }
        stage('test') {
            steps {
                script {
                    captureAndSendErrorLogs('test') {
                        sh 'mvn test'
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    captureAndSendErrorLogs('deploy') {
                        echo 'deploying'
                    }
                }
            }
        }
    }
    post {
        failure {
            script {
                // Notify failure without capturing additional logs here since already captured per stage
                emailext(
                    to: env.RECIPIENTS,
                    subject: "Jenkins Build Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} has failed. Check the individual stage logs for details."
                )
            }
        }
    }
}
def captureAndSendErrorLogs(stageName, Closure body) {
    def logBuffer = new ByteArrayOutputStream()
    def logStream = new PrintStream(logBuffer)

    try {
        // Redirect stdout to log buffer
        def originalOut = System.out
        System.setOut(logStream)
        try {
            body()
        } finally {
            System.setOut(originalOut)
        }
    } catch (Exception e) {
        // Get error log from buffer
        def errorLog = logBuffer.toString()
        // Truncate log if too large
        if (errorLog.length() > 10000) {
            errorLog = errorLog.take(10000) + "\n... [log truncated]"
        }
        
        // Send email with the error log
        emailext(
            to: env.RECIPIENTS,
            subject: "Jenkins Stage Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${stageName}",
            body: "Stage '${stageName}' in build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed.\n\nError Log:\n\n${errorLog}"
        )

        throw e // Rethrow to mark stage as failed
    } finally {
        logStream.close()
    }
}
