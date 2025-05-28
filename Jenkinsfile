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
        GIT_APP_REPO = 'https://github.com/Samir-Kesare/KEDA.git'
        GIT_KEDA_REPO = 'https://github.com/Samir-Kesare/DevOps-Optimization.git'
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
                    // Export values for envsubst
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

        stage('Apply to Kubernetes') {
            steps {
                sh "kubectl apply -f scaledobject.yaml -n ${params.NAMESPACE}"
            }
        }

        stage('Push to KEDA Repo') {
            steps {
                dir('keda-updated') {
                    script {
                        git url: "${env.GIT_KEDA_REPO}", branch: 'main', credentialsId: 'git-creds'
                        sh """
                            cp ../scaledobject.yaml ./scaled-objects/${params.SCALED_OBJECT_NAME}.yaml
                            git config user.name "jenkins"
                            git config user.email "jenkins@yourcompany.com"
                            git add .
                            git commit -m "Update ScaledObject for ${params.SERVICE_NAME} (${params.OPCO_NAME})"
                            git push origin main
                        """
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
