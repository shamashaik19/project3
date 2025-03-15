pipeline {
    agent any

    environment {
        MAVEN_HOME = tool name: 'maven', type: 'maven'
        MAVEN_CMD = "${MAVEN_HOME}/bin/mvn"
        DOCKER_CMD = "/usr/bin/docker"
        TAG_NAME = "3.0"
    }

    stages {
        stage('Prepare Environment') {
            steps {
                echo 'Initializing all variables...'
            }
        }

        stage('Git Code Checkout') {
            steps {
                script {
                    try {
                        echo 'Checking out the code from Git repository...'
                        git 'https://github.com/shamashaik19/project3.git'
                    } catch (Exception e) {
                        echo 'Exception occurred in Git Code Checkout Stage'
                        currentBuild.result = "FAILURE"
                        emailext body: '''Dear All,
                        The Jenkins job ${JOB_NAME} has failed. Please check immediately at:
                        ${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} failed', to: 'shamashaik.eee.rymec@gmail.com'
                    }
                }
            }
        }

        stage('Build the Application') {
            steps {
                echo "Cleaning, Compiling, Testing, Packaging..."
                sh "${MAVEN_CMD} clean package"
            }
        }

        stage('Publish Test Reports') {
            steps {
                publishHTML([
                    allowMissing: false, 
                    alwaysLinkToLastBuild: false, 
                    keepAll: false, 
                    reportDir: 'target/surefire-reports', 
                    reportFiles: 'index.html', 
                    reportName: 'HTML Report'
                ])
            }
        }

        stage('Containerize the Application') {
            steps {
                echo 'Creating Docker image...'
                sh "${DOCKER_CMD} build -t shamashaik19/insure-me:${TAG_NAME} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing the Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'dock-password', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | ${DOCKER_CMD} login -u $DOCKER_USER --password-stdin"
                    sh "${DOCKER_CMD} push shamashaik19/insure-me:${TAG_NAME}"
                }
            }
        }

        stage('Configure and Deploy to Test Server') {
            steps {
                echo 'Deploying application using Ansible...'
                ansiblePlaybook(
                    become: true,
                    credentialsId: 'ansible-key',
                    disableHostKeyChecking: true,
                    installation: 'ansible',
                    inventory: '/etc/ansible/hosts',
                    playbook: 'ansible-playbook.yml'
                )
            }
        }
    }
}
