pipeline {
    agent any

    parameters {
        string(name: 'SERVICE_NAME', defaultValue: '', description: 'Service Name from Jira')
        string(name: 'MIN_REPLICAS', defaultValue: '', description: 'Minimum replicas')
        string(name: 'MAX_REPLICAS', defaultValue: '', description: 'Maximum replicas')
        string(name: 'THRESHOLD', defaultValue: '', description: 'Threshold value')
        string(name: 'NAMESPACE', defaultValue: 'default', description: 'Namespace')
        string(name: 'OPCO_NAME', defaultValue: '', description: 'OpCo Name')
        string(name: 'SCALED_OBJECT_NAME', defaultValue: '', description: 'Scaled Object Name (optional)')
    }

    environment {
        GIT_APP_REPO = 'https://github.com/Samir-Kesare/DevOps-Optimization.git'
        GIT_KEDA_REPO = 'https://github.com/Samir-Kesare/KEDA.git'
    }

    stages {
        stage('Validate Params') {
            steps {
                script {
                    // If SCALED_OBJECT_NAME is empty, create default value from SERVICE_NAME
                    if (!params.SCALED_OBJECT_NAME?.trim()) {
                        env.SCALED_OBJECT_NAME = "${params.SERVICE_NAME}-keda-scaledobject"
                    } else {
                        env.SCALED_OBJECT_NAME = params.SCALED_OBJECT_NAME
                    }

                    echo "Received params:"
                    echo "SERVICE_NAME: ${params.SERVICE_NAME}"
                    echo "MIN_REPLICAS: ${params.MIN_REPLICAS}"
                    echo "MAX_REPLICAS: ${params.MAX_REPLICAS}"
                    echo "THRESHOLD: ${params.THRESHOLD}"
                    echo "NAMESPACE: ${params.NAMESPACE}"
                    echo "OPCO_NAME: ${params.OPCO_NAME}"
                    echo "SCALED_OBJECT_NAME: ${env.SCALED_OBJECT_NAME}"
                }
            }
        }

        stage('Substitute Variables in Template') {
            steps {
                script {
                    sh """
                        export SERVICE_NAME=${params.SERVICE_NAME}
                        export MIN_REPLICAS=${params.MIN_REPLICAS}
                        export MAX_REPLICAS=${params.MAX_REPLICAS}
                        export THRESHOLD=${params.THRESHOLD}
                        export NAMESPACE=${params.NAMESPACE}
                        export SCALED_OBJECT_NAME=${env.SCALED_OBJECT_NAME}
                        envsubst < scaledobject-template.yaml > scaledobject.yaml
                    """
                }
            }
        }

        stage('Clone KEDA Repo and Push Changes') {
            steps {
                script {
                    sh "rm -rf keda-repo"
                    dir('keda-repo') {
                        git url: "${env.GIT_KEDA_REPO}", branch: 'main', credentialsId: 'git-creds'
                        sh """
                            mkdir -p scaled-objects
                            cp ../scaledobject.yaml scaled-objects/${env.SCALED_OBJECT_NAME}.yaml
                            git config user.name "jenkins"
                            git config user.email "jenkins@yourcompany.com"
                            git add .
                            git commit -m "Update ScaledObject for ${params.SERVICE_NAME} (${params.OPCO_NAME})" || echo "No changes to commit"
                        """

                        withCredentials([usernamePassword(credentialsId: 'git-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                            sh """
                                git push https://${GIT_USER}:${GIT_TOKEN}@github.com/Samir-Kesare/KEDA.git main
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ ScaledObject deployed and pushed successfully."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}

