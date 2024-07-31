pipeline {
    agent any

    environment {
        SSH_KEY_ID = '2ad146f1-169e-4276-a190-d2d1b1d27f9f'
        DOCKER_VM_IP = '192.168.0.18'
        REPO_URL = 'git@github.com:Marwajouili/newAddons.git'
        CLONE_DIR = '/home/vagrant/newAddons'
        CONTAINER_NAME = 'odoo_docker-web-1'
        ADDONS_PATH = '/opt/odoo/custom_addons'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the Jenkinsfile and repository contents
                    checkout scm
                }
            }
        }

        stage('Test SSH Connection') {
            steps {
                script {
                    sshagent([SSH_KEY_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no vagrant@${DOCKER_VM_IP} "echo 'SSH connection successful'"
                        """
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    sshagent([SSH_KEY_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no vagrant@${DOCKER_VM_IP} '
                        if [ ! -d ${CLONE_DIR} ]; then
                            git clone ${REPO_URL} ${CLONE_DIR}
                        else
                            cd ${CLONE_DIR} && git pull origin main
                        fi
                        '
                        """
                    }
                }
            }
        }

        stage('Copy to Odoo Container') {
            steps {
                script {
                    sshagent([SSH_KEY_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no vagrant@${DOCKER_VM_IP} '
                        docker exec -u root ${CONTAINER_NAME} mkdir -p ${ADDONS_PATH}
                        docker cp ${CLONE_DIR} ${CONTAINER_NAME}:${ADDONS_PATH}
                        docker exec ${CONTAINER_NAME} ls -l ${ADDONS_PATH}
                        '
                        """
                    }
                }
            }
        }
    }

    triggers {
        pollSCM('H/5 * * * *') // Poll the SCM every 5 minutes
    }
}
