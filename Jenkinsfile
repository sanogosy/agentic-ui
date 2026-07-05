pipeline {
    agent any

    tools {
        nodejs "node22"
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        APP_NAME   = "AI-AGENTIC-Angular"
        BUILD_DIR  = "dist/ai-agentic"
        ZIP_FILE   = "angular-${BUILD_NUMBER}.zip"
        NEXUS_URL  = "http://172.20.10.87:8081"
        NEXUS_REPO = "angular-raw"

        NPM_CACHE  = "/var/lib/jenkins/.npm"
        NODE_OPTIONS = "--max-old-space-size=1536"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/sanogosy/agentic-ui.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm config set cache $NPM_CACHE --global
                    npm ci --prefer-offline --no-audit --no-fund
                '''
            }
        }

        stage('Build Angular') {
            steps {
                sh 'npx ng build --configuration production --source-map=false'
            }
        }

        stage('Package Build') {
            steps {
                sh """
                    cd dist
                    zip -r ${ZIP_FILE} *
                """
            }
        }

        stage('Publish to Nexus') {
            steps {
                sh """
                    curl -u admin:adminadmin --upload-file dist/${ZIP_FILE} ${NEXUS_URL}/repository/${NEXUS_REPO}/${ZIP_FILE}
                """
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'Ansible_Controller',
                        transfers: [
                            sshTransfer(
                                execCommand: "BUILD_NUMBER=${BUILD_NUMBER} ansible-playbook /opt/playbooks/download-and-deploy-angular.yaml -i /opt/playbooks/hosts",
                                execTimeout: 120000
                            )
                        ]
                    )
                ])
            }
        }

        stage('Print Info') {
            steps {
                echo "Angular build packaged as ${ZIP_FILE}"
                echo "Uploaded to Nexus repository ${NEXUS_REPO}"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

// pipeline {
//     agent any

//     tools {
//         nodejs "node22"
//     }

//     options {
//         timeout(time: 20, unit: 'MINUTES')
//         disableConcurrentBuilds()
//         timestamps()
//     }

//     environment {
//         APP_NAME   = "AI-AGENTIC-Angular"
//         BUILD_DIR  = "dist/ai-agentic"
//         ZIP_FILE   = "angular-${BUILD_NUMBER}.zip"
//         NEXUS_URL  = "http://172.20.10.87:8081"
//         NEXUS_REPO = "angular-raw"

//         NPM_CACHE  = "/var/lib/jenkins/.npm"

//         // Limite la heap de Node pour éviter que le build ne consomme
//         // toute la RAM disponible d'un coup sur une petite instance.
//         NODE_OPTIONS = "--max-old-space-size=1536"
//     }

//     stages {

//         stage('Checkout') {
//             steps {
//                 git branch: 'master', url: 'https://github.com/sanogosy/agentic-ui.git'
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 sh '''
//                     npm config set cache $NPM_CACHE --global
//                     npm ci --prefer-offline --no-audit --no-fund
//                 '''
//             }
//         }

//         stage('Build Angular') {
//             steps {
//                 sh 'npx ng build --configuration production --source-map=false'
//             }
//         }

//         stage('Package Build') {
//             steps {
//                 sh """
//                     cd dist
//                     zip -r ${ZIP_FILE} *
//                 """
//             }
//         }

//         stage('Publish to Nexus') {
//             steps {
//                 sh """
//                     curl -u admin:adminadmin --upload-file dist/${ZIP_FILE} ${NEXUS_URL}/repository/${NEXUS_REPO}/${ZIP_FILE}
//                 """
//             }
//         }

//         stage('Deploy via Ansible') {
//             steps {
//                 sshPublisher(publishers: [
//                     sshPublisherDesc(
//                         configName: 'Ansible_Controller',
//                         transfers: [
//                             sshTransfer(
//                                 execCommand: "ansible-playbook /opt/playbooks/download-and-deploy-angular.yaml -i /opt/playbooks/hosts",
//                                 execTimeout: 120000
//                             )
//                         ]
//                     )
//                 ])
//             }
//         }

//         stage('Print Info') {
//             steps {
//                 echo "Angular build packaged as ${ZIP_FILE}"
//                 echo "Uploaded to Nexus repository ${NEXUS_REPO}"
//             }
//         }
//     }

//     post {
//         always {
//             cleanWs()
//         }
//     }
// }