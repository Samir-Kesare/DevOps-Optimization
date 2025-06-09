pipeline {
    agent any

    parameters {
        text(name: 'PAYLOAD', defaultValue: '''{
    "SERVICE_NAME": "cache-mgmt-service",
    "MIN_REPLICAS": "1",
    "MAX_REPLICAS": "2",
    "THRESHOLD": "238",
    "NAMESPACE": "default",
    "OPCO_NAME": "default-opco"
}''', description: 'Jira JSON Payload')
    }

    environment {
        GIT_APP_REPO = 'https://github.com/Samir-Kesare/DevOps-Optimization.git'
        GIT_KEDA_REPO = 'https://github.com/Samir-Kesare/KEDA.git'
    }

    stages {

        stage('Parse JSON Payload') {
            steps {
                script {
                    echo "🔍 Raw PAYLOAD: ${params.PAYLOAD}"
                    def data = readJSON text: params.PAYLOAD

                    env.SERVICE_NAME        = data.SERVICE_NAME
                    env.SCALED_OBJECT_NAME  = "${data.SERVICE_NAME}-keda-scaledobject"
                    env.MIN_REPLICAS        = data.MIN_REPLICAS
                    env.MAX_REPLICAS        = data.MAX_REPLICAS
                    env.THRESHOLD           = data.THRESHOLD
                    env.NAMESPACE           = data.NAMESPACE
                    env.OPCO_NAME           = data.OPCO_NAME

                    echo "✅ Parsed Values:"
                    echo "SERVICE_NAME: ${env.SERVICE_NAME}"
                    echo "SCALED_OBJECT_NAME: ${env.SCALED_OBJECT_NAME}"
                    echo "MIN_REPLICAS: ${env.MIN_REPLICAS}"
                    echo "MAX_REPLICAS: ${env.MAX_REPLICAS}"
                    echo "THRESHOLD: ${env.THRESHOLD}"
                    echo "NAMESPACE: ${env.NAMESPACE}"
                    echo "OPCO_NAME: ${env.OPCO_NAME}"
                }
            }
        }

        stage('Clone Template Repo') {
            steps {
                git url: "${env.GIT_APP_REPO}", branch: 'main', credentialsId: 'git-creds'
            }
        }

        stage('Substitute Variables in Template') {
            steps {
                script {
                    sh """
                        export SCALED_OBJECT_NAME="${env.SCALED_OBJECT_NAME}"
                        export SERVICE_NAME="${env.SERVICE_NAME}"
                        export MIN_REPLICAS="${env.MIN_REPLICAS}"
                        export MAX_REPLICAS="${env.MAX_REPLICAS}"
                        export THRESHOLD="${env.THRESHOLD}"
                        export NAMESPACE="${env.NAMESPACE}"
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
                            git commit -m "Update ScaledObject for ${env.SERVICE_NAME} (${env.OPCO_NAME})" || echo "No changes to commit"
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


