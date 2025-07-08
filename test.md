pipeline {
    agent any
    
    triggers {
        cron('30 9,13,17 * * *')
    }

    environment {
        SERVER_LIST = "KE:172.23.36.206"
        SSH_USER = "esbuser"
        SSH_PORT = "922"
        REMOTE_SCRIPT = "/tmp/pgpool_connections_script.sh"
        LOCAL_SCRIPT = "/app/jenkins/pgpool_connection/pgpool_connections_script.sh"
        EMAIL_RECIPIENTS = "a_samir.kesare@africa.airtel.com"
        REPORT_TIMESTAMP = "${new Date().format('yyyy-MM-dd_HH-mm-ss')}"
    }

    stages {
        stage('Parse Server List') {
            steps {
                script {
                    def servers = env.SERVER_LIST.split(',').collect { entry ->
                        def parts = entry.split(':')
                        [name: parts[0].trim(), ip: parts[1].trim()]
                    }

                    env.PARSED_SERVERS = writeJSON returnText: true, json: servers
                    echo "✅ Target Servers: ${servers.collect { "${it.name}(${it.ip})" }.join(', ')}"
                }
            }
        }

        stage('Prepare Script') {
            steps {
                sh 'sudo chmod +x $LOCAL_SCRIPT' // Ensure the script is executable
            }
        }

        stage('Distribute & Execute Script') {
            steps {
                script {
                    def servers = readJSON text: env.PARSED_SERVERS
                    def executionResults = []

                    servers.each { server ->
                        try {
                            echo "==== Processing ${server.ip} (${server.name}) ===="
                            // Distribute the script
                            sh "scp -P ${SSH_PORT} ${LOCAL_SCRIPT} ${SSH_USER}@${server.ip}:${REMOTE_SCRIPT}"
                            // Execute the script and capture the output
                            def output = sh(script: "ssh -t -p ${SSH_PORT} ${SSH_USER}@${server.ip} 'bash ${REMOTE_SCRIPT} ${server.name} ${server.ip}'", returnStdout: true).trim()
                            echo output // Print the output to the Jenkins log

                            // Collect execution result
                            executionResults.add([name: server.name, ip: server.ip, status: 'SUCCESS', output: output])
                            echo "==== Successfully processed ${server.ip} (${server.name}) ===="
                        } catch (Exception e) {
                            echo "❌ ERROR: Failed to process ${server.ip} (${server.name}): ${e.getMessage()}"
                            executionResults.add([name: server.name, ip: server.ip, status: 'FAILED', error: e.getMessage()])
                        }
                    }

                    // Store execution results as JSON for email
                    env.EXECUTION_RESULTS = writeJSON returnText: true, json: executionResults
                }
            }
        }

        stage('Email Results') {
            steps {
                script {
                    // Prepare email body
                    def emailBody = "PgPool Connection Status Report - ${REPORT_TIMESTAMP}\n\n"
                    def results = readJSON text: env.EXECUTION_RESULTS
                    results.each { result ->
                        emailBody += "Server: ${result.name} (${result.ip}) - Status: ${result.status}\n"
                        if (result.status == 'SUCCESS') {
                            emailBody += "Output:\n${result.output}\n\n"
                        } else {
                            emailBody += "Error: ${result.error}\n\n"
                        }
                    }

                    // Send email with the execution report
                    emailext(
                        subject: "PgPool Connection Status Report - ${REPORT_TIMESTAMP}",
                        body: emailBody,
                        to: EMAIL_RECIPIENTS
                    )
                }
            }
        }
    }

    post {
        always {
            script {
                // Cleanup remote script if needed
                env.PARSED_SERVERS.each { server ->
                    sh "ssh -t -p ${SSH_PORT} ${SSH_USER}@${server.ip} 'rm -f ${REMOTE_SCRIPT}' || true"
                }
            }
        }
    }
}
