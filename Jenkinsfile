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

        SONAR_PROJECT_KEY = "ai-agentic-angular"
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

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.sources=src \
                              -Dsonar.exclusions=**/node_modules/**,**/dist/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package Build') {
            steps {
                sh """
                    cd dist/ai-agentic/browser
                    zip -r ../../${ZIP_FILE} *
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