pipeline {
    agent any

    parameters {
        string(name: 'SCALED_OBJECT_NAME', defaultValue: 'cache-mgmt-service-keda-scaledobject', description: 'ScaledObject name')
        string(name: 'SERVICE_NAME', defaultValue: 'cache-mgmt-service', description: 'Target Deployment name')
        string(name: 'MIN_REPLICAS', defaultValue: '1', description: 'Minimum replica count')
        string(name: 'MAX_REPLICAS', defaultValue: '1', description: 'Maximum replica count')
        string(name: 'THRESHOLD', defaultValue: '238', description: 'Threshold for Prometheus metric')
        string(name: 'NAMESPACE', defaultValue: 'default', description: 'Kubernetes namespace')
        string(name: 'OPCO_NAME', defaultValue: 'default-opco', description: 'OPCO Name (for tagging/logging purposes)')
    }

    environment {
        GIT_APP_REPO = 'https://github.com/Samir-Kesare/DevOps-Optimization.git'
        GIT_KEDA_REPO = 'https://github.com/Samir-Kesare/KEDA.git'
    }

    stages {
        stage('Clone Template Repo') {
            steps {
                git url: "${env.GIT_APP_REPO}", branch: 'main', credentialsId: 'git-creds'
            }
        }

        stage('Substitute Variables in Template') {
            steps {
                script {
                    sh """
                        export SCALED_OBJECT_NAME=${params.SCALED_OBJECT_NAME}
                        export SERVICE_NAME=${params.SERVICE_NAME}
                        export MIN_REPLICAS=${params.MIN_REPLICAS}
                        export MAX_REPLICAS=${params.MAX_REPLICAS}
                        export THRESHOLD=${params.THRESHOLD}
                        export NAMESPACE=${params.NAMESPACE}
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
                            cp ../scaledobject.yaml scaled-objects/${params.SCALED_OBJECT_NAME}.yaml
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
