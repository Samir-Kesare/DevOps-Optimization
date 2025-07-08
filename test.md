pipeline {
    agent any

    parameters {
        choice(name: 'env', choices: ['prod', 'uat'], description: 'Please Select Environment')
        choice(name: 'OPCO_NAME', choices: ['ug', 'ng', 'zm', 'cd', 'cg', 'ga', 'mg', 'mw', 'ne', 'sc', 'td', 'tz', 'ke', 'rw', 'ngpsb', 'ngpsbloadtest'], description: 'Select OPCO Name')
        choice(name: 'keda', choices: ['enabled', 'disabled', 'toggle'], description: 'Keda Status')
        choice(name: 'service', choices: ['collective', 'ds2', 'cms-backend-service', 'cms-backend-ui'], description: 'Select Service')
        choice(name: 'namespace', choices: ['default', 'mfs', 'maestro'], description: 'Select Namespace')

        string(name: 'minReplicaCount', defaultValue: '1', description: 'Minimum Replica Count (used if toggle selected)')
        string(name: 'maxReplicaCount', defaultValue: '5', description: 'Maximum Replica Count (used if toggle selected)')
        string(name: 'replica', defaultValue: '1', description: 'Replica Count (used if toggle selected)')
        string(name: 'threshold', defaultValue: '80', description: 'Threshold Value (used if toggle selected)')
    }

    environment {
        SSH_USER = 'esbuser'
        SSH_PORT = '922'
        GIT_REPO = 'https://bitbucket.airtel.africa/bitbucket/scm/atlaf/keda.git'
        GIT_USER = 'jenkins-africa@noreply.airtel.africa'
        GIT_USERNAME = 'Jenkins'
    }

    stages {
        stage('Extract Jenkins User') {
            steps {
                script {
                    def logPath = "${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}/log"
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

        stage('Preparation') {
            steps {
                script {
                    echo "Selected environment: ${params.env}"
                    echo "Selected OPCO: ${params.OPCO_NAME}"
                    echo "Selected KEDA option: ${params.keda}"
                    echo "Service: ${params.service}"
                    echo "Namespace: ${params.namespace}"

                    if (params.keda == 'toggle') {
                        echo "Toggle Parameters:"
                        echo "  Min Replica Count: ${params.minReplicaCount}"
                        echo "  Max Replica Count: ${params.maxReplicaCount}"
                        echo "  Replica Count: ${params.replica}"
                        echo "  Threshold: ${params.threshold}"
                    } else {
                        echo "Skipping toggle parameters — not required."
                    }
                }
            }
        }

        stage('Deploy Keda Configuration') {
            steps {
                script {
                    if (params.keda == 'enabled') {
                        echo "✅ Enabling KEDA for ${params.service}"
                        // Add your enabling logic
                    } else if (params.keda == 'disabled') {
                        echo "❌ Disabling KEDA for ${params.service}"
                        // Add your disabling logic
                    } else if (params.keda == 'toggle') {
                        echo "🔁 Applying KEDA toggle config"

                        // Clone repo
                        sh "git clone ${env.GIT_REPO} keda-repo"

                        dir('keda-repo') {
                            def kedaFile = "${params.OPCO_NAME}-keda-scaler/${params.namespace}/${params.service}_keda_scaledobject.yaml"

                            if (!fileExists(kedaFile)) {
                                error("❗ YAML file does not exist: ${kedaFile}")
                            }

                            sh """
                                sed -i 's/minReplicaCount: .*/minReplicaCount: ${params.minReplicaCount}/' ${kedaFile}
                                sed -i 's/maxReplicaCount: .*/maxReplicaCount: ${params.maxReplicaCount}/' ${kedaFile}
                                sed -i 's/replicas: .*/replicas: ${params.replica}/' ${kedaFile}
                                sed -i 's/value: \".*\"/value: \"${params.threshold}\"/' ${kedaFile}
                            """

                            // Git Push
                            def branch = "feature/${params.OPCO_NAME}-${env.BUILD_ID}"

                            sh """
                                git config user.name '${env.CLEAN_USER_ID}'
                                git config user.email '${env.CLEAN_USER_ID}@airtel.africa'
                                git checkout -b ${branch}
                                git add ${kedaFile}
                                git commit -m "Updated ScaledObject for ${params.service} (${params.OPCO_NAME}) by ${env.CLEAN_USER_ID}"
                                git push origin ${branch}
                            """

                            echo "✅ Pushed changes to branch: ${branch}"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ KEDA pipeline completed successfully."
        }
        failure {
            echo "❌ KEDA pipeline failed."
        }
    }
}

