pipeline {
    agent any
 
    environment {
        BASE_DIR       = '/app/jenkins/workspace/Infra-automation/Development/DevOps_Automation/Password_Expiry_Notification/password_exp'
        INVENTORY_FILE = "${BASE_DIR}/inventory.ini"
        PLAYBOOK       = "${BASE_DIR}/check_expiry.yaml"
        WARNING_FILE   = "/app/jenkins/workspace/Infra-automation/Development/DevOps_Automation/Password_Expiry_Notification/warning_summary.txt"
        GIT_REPO       = 'https://bitbucket.airtel.africa/bitbucket/scm/doc/devopscore_ansible.git'
        SENDER_EMAIL   = 'jenkins-africa@noreply.airtel.africa'
    }
 
    stages {
        stage('Clone Repository') {
            steps {
                echo '🔁 Cloning the repository...'
                git url: "${GIT_REPO}", branch: 'feature', credentialsId: 'jenkins-admin'
            }
        }
 
        stage('List Directory Contents') {
            steps {
                echo '📁 Listing contents of the Ansible directory...'
                sh "ls -la ${BASE_DIR}"
            }
        }
 
        stage('Prepare') {
            steps {
                echo '🧹 Cleaning up previous warning file...'
                sh "rm -f ${WARNING_FILE} || true"
            }
        }
 
        stage('Run Ansible Playbook') {
            steps {
                echo '🚀 Running Ansible playbook...'
                sh '''
                    set +e
 
                    ansible-playbook -i ${INVENTORY_FILE} -f 5 ${PLAYBOOK}
                    RC=$?
 
                    echo "🔍 Ansible exited with code $RC"
 
                    if [ $RC -eq 0 ]; then
                        echo "✅ Playbook ran successfully."
                        exit 0
                    elif [ $RC -eq 4 ]; then
                        echo "⚠️ Warning: Some hosts were unreachable. Continuing pipeline."
                        exit 0
                    else
                        echo "❌ Ansible failed with exit code $RC"
                        exit $RC
                    fi
                '''
            }
        }
 
        stage('Notify if Warnings Exist') {
            steps {
                script {
                    echo "📩 Checking for warnings...${WARNING_FILE}"
                    def filePath = "${WARNING_FILE}"
                    if (fileExists(filePath)) {
                        def payload = readFile(filePath).trim()
                        echo "📤 Sending warning email with content:"
                        echo payload
                        emailext (
                            subject: "Password_Expiry_Notification",
                            body: "Warnings found in the playbook execution:\n\n${payload}",
                            to: "a_samir.kesare@africa.airtel.com",
                            from: "${SENDER_EMAIL}"
                        )
                    } else {
                        echo "✅ No warnings found. Skipping notification."
                    }
                }
            }
        }
    }
 
    post {
        failure {
            script {
                echo '❌ Build failed. Sending failure notification...'
                emailext (
                    subject: "Build Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "The Jenkins job has failed. Please check the console output at: ${env.BUILD_URL}",
                    to: "a_samir.kesare@africa.airtel.com,a_naveen.verma@africa.airtel.com",
                    from: "${SENDER_EMAIL}"
                )
            }
        }
    }
}
