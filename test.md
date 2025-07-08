pipeline {
    agent any

    parameters {
        choice(name: 'env', choices: ['prod', 'uat'], description: 'Please Select Environment')

        choice(name: 'OPCO_NAME', choices: ['ug', 'ng', 'zm', 'cd', 'cg', 'ga', 'mg', 'mw', 'ne', 'sc', 'td', 'tz', 'ke', 'rw', 'ngpsb', 'ngpsbloadtest'], description: 'Select OPCO Name')

        choice(name: 'keda', choices: ['enabled', 'disabled'], description: 'Keda Status')

        extendedChoice(
            name: 'namespace',
            type: 'PT_CHECKBOX',
            multiSelectDelimiter: ',',
            description: 'Select Namespaces',
            value: 'default,mfs,maestro'
        )

        extendedChoice(
            name: 'service',
            type: 'PT_CHECKBOX',
            multiSelectDelimiter: ',',
            description: 'Select Services',
            value: 'collective,ds2,cms-backend-service,cms-backend-ui,cm,cust-profile,identity-verification,Notification-service,am-profile,customer-status,retailer-gateway,risk-eval,abtest,adapter-service,airtel-payment-options,async-report-service,bfe-gateway,Change-plan-service,cache-mgmt-service,data-service,dnd-service,email-notification-consumer,email-notifications,enterprise-service,esb-consumer-service,esb-logging-consumer-service,esb-logging-producer-service,external-gateway-service,hbb-service,hr-directory,loan-service,multi-datasource-service,nconsumer,network-service,nproducer,number-management-service,ods-data-service,pg-services,product-catalog,recharge-service-new,regulatory-service,reporting-service,ringback-tones,rscheduler-service,scheduler-service,sms-notifications,subscriber-consent,subscriber-product,subscriber-profile,subscriber-transaction-new,user-service,vas-service,vass-service,voucher-service,bonus-service,Subscriber history,esb-graph-service,Chat-notification-service,subscriber-device,customer-onboarding-service,customer-profile-service,service-provisioning-platform,cacq-cm,qna,subscriber-balance,hbb-mail-notification-service,chat-notification-service,esb-subscriber-device,bfe-gateway-service,esb-edge-service,rubik-case-management,rubik-event-api,case-management-bgservice'
        )
    }

    stages {
        stage('Preparation') {
            steps {
                script {
                    cleanWs()
                    echo "Cloning repository..."
                    sh 'rm -rf keda && git clone https://bitbucket.airtel.africa/bitbucket/scm/atlaf/keda.git'

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

                    if (params.env == 'uat') {
                        env.k8sserver = uatIP[params.OPCO_NAME]
                    } else if (params.env == 'prod') {
                        env.k8sserver = prodIP[params.OPCO_NAME]
                    }

                    echo "🌐 Selected K8s Master IP: ${env.k8sserver}"
                    echo "Selected Namespaces: ${params.namespace}"
                    echo "Selected Services: ${params.service}"
                }
            }
        }

        stage('Deploy Keda Configuration') {
            steps {
                script {
                    def namespaces = params.namespace.split(',')
                    def services = params.service.split(',')

                    for (ns in namespaces) {
                        for (svc in services) {
                            echo "🚀 Processing service '${svc}' in namespace '${ns}'"

                            if (params.keda == 'enabled') {
                                echo "✅ Enabling KEDA for ${svc}"
                                // Add enabling logic here
                            } else if (params.keda == 'disabled') {
                                echo "❌ Disabling KEDA for ${svc}"
                                // Add disabling logic here
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up...'
            cleanWs()
        }
    }
}
