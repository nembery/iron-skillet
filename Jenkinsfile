switch (env.BRANCH_NAME) {
    case '90dev':
        PANOS_VERSION_IP_ID        = 'PANOS_90_IP'
        break
    case '81dev':
        PANOS_VERSION_IP_ID        = 'PANOS_81_IP'
        break
    case '80dev':
        PANOS_VERSION_IP_ID        = 'PANOS_80_IP'
        break
    default:
        PANOS_VERSION_IP_ID        = 'PANOS_90_IP'
        break
}

pipeline {
    agent {
        docker {
            // https://github.com/nembery/SkilletLoader
            // dev ensure commit action again - touch
            image "nembery/skilletloader:dev"
            alwaysPull true
        }
    }
    environment {
        // Grab our lab rats IP and auth information from the credentials store
        PANOS_IP        = credentials("${PANOS_VERSION_IP_ID}")
        PANOS_AUTH      = credentials('PANOS_AUTH')
        PANOS_GW        = credentials('PANOS_GW')
        PANOS_MASK      = credentials('PANOS_MASK')

        // Grab our auth code to ensure the FW is fully licensed
        AUTH_CODE      = credentials('VM50_AUTH_CODE')
        // Any variables from the .meta-cnc file will be overridden with values from the environment
        // for this test, we will override the FW_NAME var and network information vars
        FW_NAME         = "test-${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        ADMINISTRATOR_USERNAME  = "${PANOS_AUTH_USR}"
        ADMINISTRATOR_PASSWORD  = "${PANOS_AUTH_PSW}"
        MGMT_TYPE               = "static"
        MGMT_IP                 = "${PANOS_IP}"
        MGMT_MASK               = "${PANOS_MASK}"
        MGMT_DG                 = "${PANOS_GW}"
    }

    stages {
        stage('Prepare PAN-OS VM') {
            steps {
                echo "Waiting for device to come online"
                sh 'wait_for_device.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                echo "Device is now ready..."
                sh 'install_license.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                echo "Device is licensed ..."
                echo "Loading latest dynamic content"
                sh 'update_dynamic_content.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW} -t content'
            }
        }
        stage('Load IronSkillet PAN-OS Snippets') {
            steps {
                echo "Loading a baseline configuration"
                sh 'load_baseline.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                echo "Baseline configuration is now complete"
                sh 'load_skillet.py ./templates/panos/snippets -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
            }
        }
        stage('Load IronSkillet PAN-OS Full Config') {
            steps {
                echo "Loading a baseline configuration"
                sh 'load_baseline.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                echo 'Importing and loading full config'
                sh 'import_and_load_full_config.py templates/panos/full -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                echo "Skillet loaded, tests complete"
            }
        }
    }
    post {
        success {
            echo 'Iron-Skillet Loaded Successfully'
            slackSend color: 'good', message: "Iron-Skillet Load for Branch ${env.BRANCH_NAME} was successful"
        }
        failure {
            echo 'Could not preform load of Iron-Skillet'
            slackSend color: 'bad', message: "Iron-Skillet Load for Branch ${env.BRANCH_NAME} failed"
        }
        always {
            echo "Build complete"
        }
    }
}
