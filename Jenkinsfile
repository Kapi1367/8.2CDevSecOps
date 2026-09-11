pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    triggers {
        pollSCM('* * * * *')
    }

    environment {
        NOTIFICATION_EMAIL = 'kapilesh.d.anandh@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kapi1367/8.2CDevSecOps.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'node --version'
                bat 'npm --version'
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    def testStatus = bat(
                        script: 'npm test',
                        returnStatus: true
                    )

                    if (testStatus == 0) {
                        emailext(
                            to: env.NOTIFICATION_EMAIL,
                            subject: "SUCCESS: Test Stage - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: """
The Run Tests stage completed successfully.

Job: ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
Status: SUCCESS
Build URL: ${env.BUILD_URL}

The Jenkins build log is attached.
""",
                            attachLog: true,
                            compressLog: false
                        )
                    } else {
                        emailext(
                            to: env.NOTIFICATION_EMAIL,
                            subject: "FAILURE: Test Stage - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: """
The Run Tests stage detected test failures.

Job: ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
Status: FAILURE
Exit code: ${testStatus}
Build URL: ${env.BUILD_URL}

The Jenkins build log is attached.
""",
                            attachLog: true,
                            compressLog: false
                        )

                        echo "Tests returned exit code ${testStatus}. The pipeline will continue."
                    }
                }
            }
        }

        stage('Generate Coverage Report') {
            steps {
                bat 'npm run coverage || exit /b 0'
            }
        }

        stage('NPM Audit (Security Scan)') {
            steps {
                script {
                    def auditStatus = bat(
                        script: 'npm audit',
                        returnStatus: true
                    )

                    if (auditStatus == 0) {
                        emailext(
                            to: env.NOTIFICATION_EMAIL,
                            subject: "SUCCESS: Security Scan - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: """
The NPM Audit security scan completed successfully.

Job: ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
Status: SUCCESS
Build URL: ${env.BUILD_URL}

The Jenkins build log is attached.
""",
                            attachLog: true,
                            compressLog: false
                        )
                    } else {
                        emailext(
                            to: env.NOTIFICATION_EMAIL,
                            subject: "FAILURE: Security Scan - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            body: """
The NPM Audit security scan found known vulnerabilities.

Job: ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
Status: FAILURE - SECURITY ISSUES FOUND
Exit code: ${auditStatus}
Build URL: ${env.BUILD_URL}

Review the attached Jenkins build log for vulnerability details.
""",
                            attachLog: true,
                            compressLog: false
                        )

                        echo "NPM Audit found vulnerabilities. This is expected for nodejs-goof."
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'DevSecOps pipeline execution completed.'
        }
    }
}
