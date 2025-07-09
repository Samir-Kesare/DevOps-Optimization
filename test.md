pipeline {
    agent any

//     triggers {
//         cron('H * * * *') // Uncomment for hourly schedule
//     }

    environment {
        SERVER_LIST = "KE:172.23.36.206,ZM:172.27.128.119"
        SSH_USER = "esbuser"
        SSH_PORT = "922"
        REMOTE_SCRIPT = "/tmp/pgpool_connections_script.sh"
        LOCAL_SCRIPT = "/app/jenkins/pgpool_connection/pgpool_connections_script.sh"
        EMAIL_RECIPIENTS = "a_samir.kesare@africa.airtel.com,a_shristi.gupta@africa.airtel.com"
    }

    stages {
        stage('Init Variables') {
            steps {
                script {
                    env.REPORT_TIMESTAMP = new Date().format('yyyy-MM-dd_HH-mm-ss')
                    env.ATTACHMENT_LIST = ''
                }
            }
        }

        stage('Parse Server List') {
            steps {
                script {
                    def servers = env.SERVER_LIST.split(',').collect { entry ->
                        def parts = entry.split(':')
                        [name: parts[0].trim(), ip: parts[1].trim()]
                    }

                    env.PARSED_SERVERS = writeJSON returnText: true, json: servers
                    echo "✅ Servers: ${servers.collect { "${it.name}(${it.ip})" }.join(', ')}"
                }
            }
        }

        stage('Execute on Servers') {
            steps {
                script {
                    def servers = readJSON text: env.PARSED_SERVERS
                    def attachments = []

                    servers.each { server ->
                        def serverName = server.name
                        def serverIP = server.ip
                        def reportFile = "pgpool_conn_report_${serverName}_${env.REPORT_TIMESTAMP}.log"
                        
                        echo "🔧 Executing on ${serverName} (${serverIP})"

                        sh """
                            scp -P ${env.SSH_PORT} ${env.LOCAL_SCRIPT} ${env.SSH_USER}@${serverIP}:${env.REMOTE_SCRIPT}
                            ssh -p ${env.SSH_PORT} ${env.SSH_USER}@${serverIP} 'chmod +x ${env.REMOTE_SCRIPT} && bash ${env.REMOTE_SCRIPT}'
                            ssh -p ${env.SSH_PORT} ${env.SSH_USER}@${serverIP} 'cat ~/Default_conn.txt' > ${reportFile}
                        """

                        attachments << reportFile
                    }

                    // Store all attachments in environment variable (comma-separated)
                    env.ATTACHMENT_LIST = attachments.join(',')
                    echo "📎 Attachment files: ${env.ATTACHMENT_LIST}"
                }
            }
        }

        stage('Send Email Report') {
            steps {
                script {
                    emailext(
                        subject: "[PGPool Report] Multi-Server Report - ${env.REPORT_TIMESTAMP}",
                        body: """
                            Hello Team,<br><br>
                            Please find the PGPool connection reports for each server attached below.<br><br>
                            Regards,<br>Jenkins
                        """,
                        to: "${env.EMAIL_RECIPIENTS}",
                        attachmentsPattern: "${env.ATTACHMENT_LIST}",
                        mimeType: 'text/html'
                    )
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
