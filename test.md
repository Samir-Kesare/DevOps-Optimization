stages {
        stage('Preparation') {
            steps {
                script {
                    // Clone the Bitbucket repository
                    sh 'git clone http://172.23.7.66:8090/path/to/your/repo.git'

                    // Define IP addresses based on environment and OPCO_NAME
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

                    // Determine the k8s server IP based on the selected environment and OPCO_NAME
                    if (params.env == 'uat') {
                        env.k8sserver = uatIP[params.OPCO_NAME]
                    } else if (params.env == 'prod') {
                        env.k8sserver = prodIP[params.OPCO_NAME]
                    }
                }
            }
        }

