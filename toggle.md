pipeline {
    agent any

    parameters {
        choice(name: 'env', choices: ['prod', 'uat'], description: 'Please Select Environment')
        choice(name: 'OPCO_NAME', choices: ['ug', 'ng', 'zm', 'cd', 'cg', 'ga', 'mg', 'mw', 'ne', 'sc', 'td', 'tz', 'ke', 'rw', 'ngpsb', 'ngpsbloadtest'], description: 'Select OPCO Name')
        string(name: 'keda', defaultValue: 'toggle', description: 'Keda Status')
        choice(name: 'service', choices: ['collective', 'ds2', 'cms-backend-service', 'cms-backend-ui'], description: 'Select Service')
        choice(name: 'namespace', choices: ['default', 'mfs', 'maestro'], description: 'Select Namespace')
        string(name: 'minReplicaCount', defaultValue: '1', description: 'Minimum Replica Count')
        string(name: 'maxReplicaCount', defaultValue: '5', description: 'Maximum Replica Count')
        string(name: 'replica', defaultValue: '1', description: 'Replica Count')
        string(name: 'threshold', defaultValue: '80', description: 'Threshold Value')
    }

    environment {
        SSH_USER = 'esbuser'
        SSH_PORT = '922'
        GIT_REPO = 'https://bitbucket.airtel.africa/bitbucket/scm/atlaf/keda.git'
    }

    stages {
        stage('Extract Jenkins User') {
            steps {
                cleanWs()
                script {
                    def logPath = "${JENKINS_HOME}/jobs/Infra-automation/jobs/Development/jobs/DevOps_Automation/jobs/keda_toggle_test/builds/${BUILD_ID}/log"
                    sh """
                        echo "Reading log: ${logPath}"
                        cat ${logPath} > log.txt || true
                        user_id=\$(cat log.txt | grep -i "Started" | sed "s@Started by user @@")
                        echo "\$user_id" > user_id.txt
                    """
                    def rawUserId = readFile('user_id.txt').trim()
                    env.CLEAN_USER_ID = rawUserId.find(/\d{6,10}$/) ?: "unknown-user"
                    echo "✅ Clean USER_ID: ${env.CLEAN_USER_ID}"
                }
            }
        }

        stage('Set Kubernetes Master IP') {
            steps {
                script {
                    def prodIP = [
                        ke: '172.23.7.102', mw: '172.26.128.101', ga: '172.25.118.125', mg: '172.25.128.181',
                        ne: '172.26.193.149', cg: '172.25.64.209', zm: '172.27.128.119', cd: '172.26.38.126',
                        rw: '172.27.193.85', ngpsb: '172.24.30.11', td: '172.25.49.170', sc: '172.25.192.172',
                        ng: '172.24.6.215', ug: '172.27.98.166', tz: '172.27.0.142'
                    ]
                    def uatIP = [
                        cd: '172.26.18.29', cg: '172.25.67.229', ga: '172.25.119.191', ke: '172.23.36.206',
                        mg: '172.25.131.133', mw: '172.26.146.190', ne: '172.26.211.41', ng: '172.24.35.202',
                        rw: '172.27.210.164', sc: '172.25.195.226', td: '172.25.49.191', tz: '172.27.18.120',
                        ug: '172.27.82.150', zm: '172.27.146.167', ngpsb: '172.24.31.11', ngpsbloadtest: '172.24.30.224'
                    ]

                    def selectedMap = params.env == 'prod' ? prodIP : uatIP

                    if (!selectedMap.containsKey(params.OPCO_NAME)) {
                        error "❌ OPCO '${params.OPCO_NAME}' is not defined for '${params.env}' environment"
                    }

                    env.k8sserver = selectedMap[params.OPCO_NAME]
                    echo "🌐 Selected Kubernetes master IP: ${env.k8sserver}"
                }
            }
        }

        stage('Keda Toggle') {
            steps {
                script {
                    echo "🔁 Applying KEDA toggle config"

                    sh "git clone ${env.GIT_REPO} keda-repo"

                    dir('keda-repo') {
                        def kedaFile = "${params.OPCO_NAME}-keda-scaler/${params.namespace}/${params.service}_keda_scaledobject.yaml"
                        def kedapath = "${pwd()}/${kedaFile}"

                        echo "🔍 Original YAML:"
                        sh "cat ${kedaFile}"

                        sh """
                            sed -i 's/^  minReplicaCount: .*/  minReplicaCount: ${params.minReplicaCount}/' ${kedaFile}
                            sed -i 's/^  maxReplicaCount: .*/  maxReplicaCount: ${params.maxReplicaCount}/' ${kedaFile}
                            sed -i 's/^  replicas: .*/  replicas: ${params.replica}/' ${kedaFile}
                            sed -i '/type: prometheus/,/threshold:/s/threshold: .*/threshold: \\'${params.threshold}\\'/' ${kedaFile}
                        """

                        echo "📝 Updated YAML:"
                        sh "cat ${kedaFile}"

                        sh "scp -P ${env.SSH_PORT} ${kedapath} ${env.SSH_USER}@${env.k8sserver}:/tmp/${params.service}_keda_scaledobject.yaml"
                        sh "ssh -p ${env.SSH_PORT} ${env.SSH_USER}@${env.k8sserver} 'kubectl apply -f /tmp/${params.service}_keda_scaledobject.yaml -n ${params.namespace} --dry-run=client'"

                        def branch = "feature/${params.OPCO_NAME}-${env.BUILD_ID}"
                        sh """
                            git config user.name '${env.CLEAN_USER_ID}'
                            git config user.email '${env.CLEAN_USER_ID}@airtel.africa'
                            git checkout -b ${branch}
                            git add ${kedaFile}
                            git commit -m "Updated ScaledObject for ${params.service} (${params.OPCO_NAME}) by ${env.CLEAN_USER_ID}"
                            git push --dry-run origin ${branch}
                        """

                        echo "✅ Dry-run git push to branch: ${branch}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "✅ KEDA pipeline completed successfully."
        }
        failure {
            echo "❌ KEDA pipeline failed."
        }
    }
}
