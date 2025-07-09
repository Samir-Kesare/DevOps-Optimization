pipeline {
    agent any

//    triggers {
//        cron('H * * * *') // Runs every hour at a random minute
//    }

    environment {
        SERVER_LIST = "KE:172.23.36.206,ZM:"
        SSH_USER = "esbuser"
        SSH_PORT = "922"
        REMOTE_SCRIPT = "/tmp/pgpool_connections_script.sh"
        LOCAL_SCRIPT = "/app/jenkins/pgpool_connection/pgpool_connections_script.sh"
        EMAIL_RECIPIENTS = "a_samir.kesare@africa.airtel.com, a_shristi.gupta@africa.airtel.com"
    }

    stages {

        stage('Init Report Variables') {
            steps {
                script {
                    // Move dynamic values to script scope
                    env.REPORT_TIMESTAMP = new Date().format('yyyy-MM-dd_HH-mm-ss')
                    env.REPORT_LOG = "pgpool_conn_report_${env.REPORT_TIMESTAMP}.log"
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
                    echo "✅ Target Servers: ${servers.collect { "${it.name}(${it.ip})" }.join(', ')}"
                }
            }
        }

        stage('Copy and Execute Script') {
            steps {
                script {
                    def servers = readJSON text: env.PARSED_SERVERS

                    servers.each { server ->
                        echo "🔧 Working on ${server.name} (${server.ip})"

                        sh """
                            scp -P ${env.SSH_PORT} ${env.LOCAL_SCRIPT} ${env.SSH_USER}@${server.ip}:${env.REMOTE_SCRIPT}
                            ssh -p ${env.SSH_PORT} ${env.SSH_USER}@${server.ip} 'chmod +x ${env.REMOTE_SCRIPT} && bash ${env.REMOTE_SCRIPT}'
                            ssh -p ${env.SSH_PORT} ${env.SSH_USER}@${server.ip} 'cat /tmp/Default_conn.txt' > ${env.REPORT_LOG}
                        """
                    }
                }
            }
        }

        stage('Send Email Report') {
            steps {
                script {
                    emailext(
                        subject: "[PGPool Report] Hourly Connection Report - ${env.REPORT_TIMESTAMP}",
                        body: "Hello Team,<br><br>Please find the attached PGPool connection report.<br><br>Regards,<br>Jenkins",
                        to: "${env.EMAIL_RECIPIENTS}",
                        attachmentsPattern: "${env.REPORT_LOG}",
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

