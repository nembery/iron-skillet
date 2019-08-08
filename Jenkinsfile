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
        PANOS_90_IP     = credentials('PANOS_90_IP')
        PANOS_AUTH      = credentials('PANOS_AUTH')
        PANOS_GW        = credentials('PANOS_GW')
        PANOS_MASK      = credentials('PANOS_MASK')
        // Any variables from the .meta-cnc file will be overridden with values from the environment
        // for this test, we will override the FW_NAME var
        FW_NAME         = "test-90-${env.BUILD_NUMBER}"
        ADMINISTRATOR_USERNAME  = "${PANOS_AUTH_USR}"
        ADMINISTRATOR_PASSWORD  = "${PANOS_AUTH_PSW}"
        MGMT_TYPE               = "static"
        MGMT_IP                 = "${PANOS_90_IP}"
        MGMT_MASK               = "${PANOS_MASK}"
        MGMT_DG                 = "${PANOS_GW}"

    }
    stages {
        stage('load_full_config') {
            steps {
                sh 'echo "Waiting for device to come online"'
                sh 'wait_for_device.py -i ${PANOS_90_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                sh 'echo "Device is now ready..."'
                sh 'echo "Loading latest dynamic content"'
                sh 'update_dynamic_content.py -i ${PANOS_90_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW} -t content'
                sh 'echo "Loading a baseline configuration"'
                sh 'load_baseline.py -i ${PANOS_90_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                sh 'echo "Baseline configuration is not complete"'
                sh 'load_skillet.py ./templates/panos/snippets -i ${PANOS_90_IP} -u ${PANOS_AUTH_USR} -p ${PANOS_AUTH_PSW}'
                sh 'echo "Skillet loaded, tests complete"'
            }
        }
    }
    post {
        success {
            echo 'Iron-Skillet Loaded Successfully'
        }
        failure {
            echo 'Could not preform load of Iron-Skillet'
        }
        always {
            echo "Build complete"
        }
    }
}
