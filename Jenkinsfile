
if (env.BRANCH_NAME == "90dev") {
    def PANOS_IP = "PANOS_90_IP"
}

if (env.BRANCH_NAME == "81dev") {
    def PANOS_IP = "PANOS_81_IP"
}

if (env.BRANCH_NAME == "80dev") {
    def PANOS_IP = "PANOS_80_IP"
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
        PANOS_IP        = credentials("${PANOS_IP}")
        PANOS_AUTH      = credentials("PANOS_AUTH")
        PANOS_GW        = credentials("PANOS_GW")
        PANOS_MASK      = credentials("PANOS_MASK")
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
        stage("Prepare PAN-OS VM") {
            steps {
                echo "Waiting for device to come online"
                sh "wait_for_device.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}"
                echo "Device is now ready..."
                echo "Loading latest dynamic content"
                sh "update_dynamic_content.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW} -t content"
            }
        }
        stage("Load IronSkillet PAN-OS Snippets") {
            steps {
                echo "Loading a baseline configuration"
                sh "load_baseline.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}"
                echo "Baseline configuration is now complete"
                sh "load_skillet.py ./templates/panos/snippets -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}"
            }
        }
        stage("Load IronSkillet PAN-OS Full Config") {
            steps {
                echo "Loading a baseline configuration"
                sh "load_baseline.py -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}"
                echo "Importing and loading full config"
                sh "import_and_load_full_config.py templates/panos/full -i ${PANOS_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}"
                echo "Skillet loaded, tests complete"
            }
        }
    }
    post {
        success {
            echo "Iron-Skillet Loaded Successfully"
            slackSend color: "good", message: "Iron-Skillet Load for Branch ${env.BRANCH_NAME} was successful"
        }
        failure {
            echo "Could not preform load of Iron-Skillet"
            slackSend color: "bad", message: "Iron-Skillet Load for Branch ${env.BRANCH_NAME} failed"
        }
        always {
            echo "Build complete"
        }
    }
}
