pipeline {
    agent any

    environment {
        GIT_TEMPLATE_REPO = 'https://github.com/Samir-Kesare/DevOps-Optimization.git'
        GIT_KEDA_REPO     = 'https://github.com/Samir-Kesare/KEDA.git'
        BRANCH_NAME       = 'main'
    }

    parameters {
        string(name: 'SCALED_OBJECT_NAME', defaultValue: 'cache-mgmt-service-keda-scaledobject', description: 'Name of the ScaledObject YAML file')
        string(name: 'SERVICE_NAME', defaultValue: 'cache-mgmt-service', description: 'Name of the Kubernetes service to scale')
        string(name: 'MIN_REPLICAS', defaultValue: '1', description: 'Minimum number of replicas')
        string(name: 'MAX_REPLICAS', defaultValue: '10', description: 'Maximum number of replicas')
        string(name: 'THRESHOLD', defaultValue: '50', description: 'Trigger threshold')
        string(name: 'NAMESPACE', defaultValue: 'default', description: 'Namespace in which to deploy')
        string(name: 'OPCO_NAME', defaultValue: 'default-opco', description: 'OPCO name used for metadata/labeling')
    }

    stages {
        stage('Clone ScaledObject Template Repo') {
            steps {
                dir('template-repo') {
                    git url: "${env.GIT_TEMPLATE_REPO}", branch: "${env.BRANCH_NAME}"
                }
            }
        }

        stage('Generate ScaledObject YAML') {
            steps {
                script {
                    def templateFile = readFile 'template-repo/scaled-object-template.yaml'
                    def renderedYaml = templateFile
                        .replaceAll("\\{\\{SCALED_OBJECT_NAME\\}\\}", params.SCALED_OBJECT_NAME)
                        .replaceAll("\\{\\{SERVICE_NAME\\}\\}", params.SERVICE_NAME)
                        .replaceAll("\\{\\{MIN_REPLICAS\\}\\}", params.MIN_REPLICAS)
                        .replaceAll("\\{\\{MAX_REPLICAS\\}\\}", params.MAX_REPLICAS)
                        .replaceAll("\\{\\{THRESHOLD\\}\\}", params.THRESHOLD)
                        .replaceAll("\\{\\{NAMESPACE\\}\\}", params.NAMESPACE)
                        .replaceAll("\\{\\{OPCO_NAME\\}\\}", params.OPCO_NAME)

                    writeFile file: 'scaledobject.yaml', text: renderedYaml
                }
            }
        }

        stage('Clone KEDA Repo') {
            steps {
                dir('keda-repo') {
                    git url: "${env.GIT_KEDA_REPO}", branch: "${env.BRANCH_NAME}", credentialsId: 'git-creds'
                }
            }
        }

        stage('Copy and Commit ScaledObject') {
            steps {
                dir('keda-repo') {
                    script {
                        sh """
                            mkdir -p scaled-objects
                            cp ../scaledobject.yaml ./scaled-objects/${params.SCALED_OBJECT_NAME}.yaml
                            git config user.name "jenkins"
                            git config user.email "jenkins@yourcompany.com"
                            git add .
                            git commit -m "Update ScaledObject for ${params.SERVICE_NAME} (${params.OPCO_NAME})" || echo "Nothing to commit"
                        """
                    }
                }
            }
        }

        stage('Push to KEDA Repo') {
            steps {
                dir('keda-repo') {
                    withCredentials([usernamePassword(credentialsId: 'git-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/Samir-Kesare/KEDA.git
                            git push origin ${env.BRANCH_NAME}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ ScaledObject YAML created and pushed successfully.'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}

