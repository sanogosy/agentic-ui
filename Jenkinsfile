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

        # Cache npm + node_modules
        NPM_CACHE = "/var/lib/jenkins/.npm"
        NODE_CACHE = "/var/lib/jenkins/node_cache"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/sanogosy/agentic-ui.git'
            }
        }

        stage('Prepare Cache') {
            steps {
                sh '''
                    mkdir -p $NODE_CACHE
                    mkdir -p node_modules

                    # Copier le cache node_modules dans le workspace
                    rsync -a $NODE_CACHE/ node_modules/ || true
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    # Activer le cache npm
                    npm config set cache $NPM_CACHE --global

                    # Installer les dépendances
                    npm ci

                    # Mettre à jour le cache node_modules
                    rsync -a node_modules/ $NODE_CACHE/
                '''
            }
        }

        stage('Build Angular') {
            steps {
                sh 'npx ng build --configuration production'
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