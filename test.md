pipeline {
    agent any

    triggers {
        cron('H * * * *') // Runs every hour at a random minute
    }

    environment {
        SERVER_LIST = "KE:172.23.36.206"
        SSH_USER = "esbuser"
        SSH_PORT = "922"
        REMOTE_SCRIPT = "/tmp/pgpool_connections_script.sh"
        LOCAL_SCRIPT = "/app/jenkins/pgpool_connection/pgpool_connections_script.sh"
        EMAIL_RECIPIENTS = "a_samir.kesare@africa.airtel.com"
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

        stage('Pars
