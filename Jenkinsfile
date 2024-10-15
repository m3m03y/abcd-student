pipeline {
    agent any

    environment {
        ZAP_CONF='resources/dast/zap'
        APP_SRC='juice-shop'
        REPORT_DIR='results'
    }

    parameters {
        string(name: 'EMAIL', defaultValue: 'email@example.com', description: 'ABC-DevSecOps user email')
        string(name: 'BRANCH', defaultValue: 'main', description: 'ABC-DevSecOps source branch')
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-token', url: 'https://github.com/m3m03y/abcd-student', branch: '${BRANCH}'
                }
            }
        }

        stage('Prepare reports space') {
            steps {
                sh '''
                    mkdir ${REPORT_DIR}
                    mkdir reports
                '''
            }
        }

        stage('[OSV-SCAN] Run scan') {
            steps {
                echo 'Starting osv scan...'
                sh '''
                    osv-scanner scan --lockfile ${APP_SRC}/package-lock.json --format json --output ${REPORT_DIR}/osv-scan-results.json || exit_code=$?
                    if [ $exit_code -ne 0 ] && [ ! -s ${REPORT_DIR}/osv-scan-results.json ]; then
                        echo "Scan failed and output file is empty!"
                        exit 1
                    else
                        echo "Scan completed successfully or results file exists."
                        exit 0
                    fi
                '''
                echo 'Uploading OSV scan report to DefectDojo'
                defectDojoPublisher(artifact: '${REPORT_DIR}/osv-scan-results.json', 
                    productName: 'Juice Shop', 
                    scanType: 'OSV Scan', 
                    engagementName: '${EMAIL}') 
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

        stage('[ZAP] Passive and active scan') {
            steps {
                echo 'Starting zap container...'
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/${ZAP_CONF}/:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "mkdir -p /zap/reports && zap.sh -cmd -addonupdate \
                        && zap.sh -cmd -addoninstall communityScripts \
                        -addoninstall pscanrulesAlpha \
                        -addoninstall pscanrulesBeta \
                        -autorun /zap/wrk/active_scan.yaml"
                    docker cp zap:/zap/reports ${REPORT_DIR}/
                '''
                echo 'Uploading ZAP scan report to DefectDojo'
                defectDojoPublisher(artifact: '${REPORT_DIR}/reports/zap_report.xml', 
                    productName: 'Juice Shop', 
                    scanType: 'ZAP Scan', 
                    engagementName: '${EMAIL}') 
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