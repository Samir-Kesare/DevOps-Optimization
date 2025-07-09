pipeline {
    agent any

//     triggers {
//         cron('H * * * *') // Optional: run every hour
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
        stage('Init') {
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

        stage('Copy & Execute Script, Generate CSV') {
            steps {
                script {
                    def servers = readJSON text: env.PARSED_SERVERS
                    def attachments = []

                    servers.each { server ->
                        def serverName = server.name
                        def serverIP = server.ip
                        def rawReport = "raw_report_${serverName}.log"
                        def csvReport = "pgpool_conn_report_${serverName}_${env.REPORT_TIMESTAMP}.csv"

                        echo "🚀 Running PGPool script on ${serverName} (${serverIP})"

                        sh """
                            scp -P ${env.SSH_PORT} ${env.LOCAL_SCRIPT} ${env.SSH_USER}@${serverIP}:${env.REMOTE_SCRIPT}
                            ssh -p ${env.SSH_PORT} ${env.SSH_USER}@${serverIP} 'chmod +x ${env.REMOTE_SCRIPT} && bash ${env.REMOTE_SCRIPT}'
                            ssh -p ${env.SSH_PORT} ${env.SSH_USER}@${serverIP} 'cat ~/Default_conn.txt' > ${rawReport}
                        """

                        // Convert raw log to CSV
                        writeFile file: csvReport, text: convertToCsv(readFile(rawReport))
                        attachments << csvReport
                    }

                    env.ATTACHMENT_LIST = attachments.join(',')
                    echo "📎 CSV Attachments: ${env.ATTACHMENT_LIST}"
                }
            }
        }

        stage('Send Email') {
            steps {
                script {
                    emailext(
                        subject: "[PGPool Report] CSV Connection Report - ${env.REPORT_TIMESTAMP}",
                        body: """
                            Hello Team,<br><br>
                            Please find attached PGPool CSV reports per server.<br><br>
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

// Utility method to convert raw report to CSV format
def convertToCsv(String raw) {
    def lines = raw.readLines()
    def output = new StringBuilder()

    // Extract summary table
    def summary = []
    def inSummary = false
    for (line in lines) {
        if (line.contains('+--------------------+')) {
            inSummary = !inSummary
            continue
        }
        if (inSummary && line.contains('|')) {
            def parts = line.split('\\|').collect { it.trim() }
            if (parts.size() >= 4 && parts[1] != "Namespace") {
                summary << "${parts[1]},${parts[2]},${parts[3]}"
            }
        }
    }
    output << "Namespace,Total Connections,Available Connections\n"
    summary.each { output << it + "\n" }

    // Extract service details
    def currentNamespace = ""
    boolean inServiceSection = false
    lines.each { line ->
        if (line.startsWith("Default Namespace")) {
            currentNamespace = "default"
            inServiceSection = true
            output << "\nNamespace: ${currentNamespace}\nCount,ServiceName\n"
        } else if (line.startsWith("am-finrep Namespace")) {
            currentNamespace = "am-finrep"
            output << "\nNamespace: ${currentNamespace}\nCount,ServiceName\n"
        } else if (inServiceSection && line.contains('#')) {
            def parts = line.split('#', 2)
            if (parts.length == 2) {
                output << "${parts[0].trim()},${parts[1].trim()}\n"
            }
        }
    }

    return output.toString()
}
