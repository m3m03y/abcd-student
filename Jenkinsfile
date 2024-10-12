pipeline {
    agent any

    environment {
        ZAP_CONF='resources/dast/zap'
        APP_SRC='juice-shop'
        REPORT_DIR='results'
    }

    parameters {
        string(name: 'EMAIL', defaultValue: 'email@example.com', description: 'ABC-DevSecOps user email')
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-token', url: 'https://github.com/m3m03y/abcd-student', branch: 'sca-implementation'
                }
            }
        }

        stage('Prepare Juice Shop') {
            steps {
                sh '''
                    docker volume create shared
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        -v shared:/juice-shop \
                        bkimminich/juice-shop
                    sleep 5
                '''
            }
        }

        stage('Prepare reports space') {
            steps {
                sh '''
                    mkdir ${REPORT_DIR}
                '''
            }
        }

        stage('[ZAP] Passive and active scan') {
            steps {
                echo 'Starting zap container...'
                echo "Workspace: ${WORKSPACE}"
                echo "ZAP Conf: ${ZAP_CONF}"
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/${ZAP_CONF}/:/zap/wrk/:rw \
                        -v ${WORKSPACE}/reports/:/zap/wrk/reports/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate \
                        && zap.sh -cmd -addoninstall communityScripts \
                        -addoninstall pscanrulesAlpha \
                        -addoninstall pscanrulesBeta \
                        -autorun /zap/wrk/active_scan.yaml"
                    docker cp zap:/zap/wrk/reports ${REPORT_DIR}/
                '''
                // echo 'Uploading ZAP scan report to DefectDojo'
                // defectDojoPublisher(artifact: '${REPORT_DIR}/reports/zap_report.xml', 
                //     productName: 'Juice Shop', 
                //     scanType: 'ZAP Scan', 
                //     engagementName: '${EMAIL}') 
            }

            post {
                always {
                    sh '''
                        docker stop zap
                        docker rm zap
                    '''
                }
            }
        }

        stage('[OSV-SCAN] Setup container') {
            steps {
                echo 'Starting osv-scan container...'
                sh '''
                    docker run -d --name osv-scan \
                        -v ${WORKSPACE}/${APP_SRC}:/src \
                        ghcr.io/google/osv-scanner \
                        --format json \
                        -L /src/package-lock.json \
                        --output /src/osv-scan-results.json
                    docker logs osv-scan
                    docker cp osv-scan:/src/osv-scan-results.json ${REPORT_DIR}/
                '''
                // echo 'Uploading OSV scan report to DefectDojo'
                // defectDojoPublisher(artifact: '${REPORT_DIR}/osv-scan-results.json', 
                //     productName: 'Juice Shop', 
                //     scanType: 'OSV Scan', 
                //     engagementName: '${EMAIL}') 
            }
            post {
                always {
                    sh '''
                        docker stop osv-scan
                        docker rm osv-scan
                    '''
                }
            }
        }

        stage('Archive results') {
            steps {
                echo 'Archiving results...'
                archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            }
        }       
   }

    post {
        always {
            sh '''
                docker stop juice-shop
                docker volume rm shared
            '''
        }
    }
}