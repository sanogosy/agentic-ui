pipeline {
    agent any

    tools {
        nodejs "node22"
    }

    environment {
        APP_NAME = "AI-AGENTIC-Angular"
        BUILD_DIR = "dist/ai-agentic"
        ZIP_FILE = "angular-${BUILD_NUMBER}.zip"
        NEXUS_URL = "http://172.20.10.87:8081"
        NEXUS_REPO = "angular-raw"
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
                    npm install -g @angular/cli
                    npm install
                '''
            }
        }

        stage('Build Angular') {
            steps {
                sh 'ng build --configuration production'
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
                                execCommand: "ansible-playbook /opt/playbooks/deploy-angular.yaml -i /opt/playbooks/hosts",
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
}
