pipeline {
    agent any

    environment {
        WORKSPACE='/home/ubuntu/abcd-student/resources'
        REPORT_DIR='results'
        SHARED_DIR='/shared'
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
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        -v ${SHARED_DIR}:/src \
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

        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v ${WORKSPACE}/:/zap/wrk/:rw \
                        -v ${WORKSPACE}/reports/:/zap/wrk/reports/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate \
                        && zap.sh -cmd -addoninstall communityScripts \
                        -addoninstall pscanrulesAlpha \
                        -addoninstall pscanrulesBeta \
                        -autorun /zap/wrk/active_scan.yaml"
                '''
            }
        }

        stage('[ZAP] Copy scan result') {
            steps {
                sh '''
                    docker cp zap:/zap/wrk/reports ${REPORT_DIR}/
                '''
            }
        }

        stage('[ZAP] Upload report to Defect Dojo') {
            steps {
                sh '''
                    echo Send report to DefectDojo from: ${EMAIL}
                '''
                defectDojoPublisher(artifact: '${REPORT_DIR}/reports/zap_report.xml', 
                    productName: 'Juice Shop', 
                    scanType: 'ZAP Scan', 
                    engagementName: '${EMAIL}') 
            }
        }

        stage('[OSV-SCAN] Setup container') {
            steps {
                sh '''
                    docker run -d --name osv-scan \
                        -v ${SHARED_DIR}:/src \
                        ghcr.io/google/osv-scanner \
                        --format json \
                        -L /src/package-lock.json \
                        --output /src/osv-scan-results.json
                '''
            }
        }

        stage('[OSV-SCAN] Copy results and create artifacts') {
            steps {
                sh '''
                    docker cp osv-scan:/src/osv-scan-results.json ${REPORT_DIR}/
                '''
            }
        }

        stage('[OSV-SCAN] Upload report to Defect Dojo') {
            steps {
                defectDojoPublisher(artifact: '${REPORT_DIR}/osv-scan-results.json', 
                    productName: 'Juice Shop', 
                    scanType: 'OSV Scan', 
                    engagementName: '${EMAIL}') 
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
                docker stop zap juice-shop osv-scan
                docker rm zap
            '''
        }
    }
}