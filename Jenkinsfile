pipeline {
    // 1. Release the agent globally so it does not stay blocked during the 'input' step
    agent none 

    tools {
        maven 'Maven 3.9.9'
        jdk 'JDK21'
    }

    environment {
        APP_NAME = 'devops-multi-env'
        TOMCAT_URL = 'http://172.31.47.158:8090'
    }

    stages {
        // 2. Group construction stages in a sequential block to use the same agent without cloning code multiple times
        stage('Build & Test & Local Deploy') {
            agent any
            stages {
                stage('Checkout') {
                    steps {
                        echo 'Downloading source code from GitHub...'
                        checkout scm
                    }
                }

                stage('Build') {
                    steps {
                        echo 'Building the application...'
                        sh 'mvn -B clean compile'
                    }
                }

                stage('Test') {
                    steps {
                        echo 'Running application tests...'
                        sh 'mvn -B test'
                    }
                }

                stage('Package') {
                    steps {
                        echo 'Creating the WAR file...'
                        sh 'mvn -B package -DskipTests'
                        sh 'test -f target/devops-multi-env.war'
                    }
                }

                stage('Deploy DEV') {
                    steps {
                        echo 'Deploying to DEV...'
                        deploy adapters: [
                            tomcat9(credentialsId: 'TomcatCreds', url: "${TOMCAT_URL}")
                        ],
                        contextPath: '/dev',
                        war: 'target/devops-multi-env.war'
                    }
                }

                stage('Verify DEV') {
                    steps {
                        sh 'curl --fail --silent --show-error --retry 5 --retry-delay 2 "${TOMCAT_URL}/dev/" -o /dev/null'
                        echo 'DEV deployment verified successfully.'
                    }
                }

                stage('Deploy SIT') {
                    steps {
                        echo 'Deploying to SIT...'
                        deploy adapters: [
                            tomcat9(credentialsId: 'TomcatCreds', url: "${TOMCAT_URL}")
                        ],
                        contextPath: '/sit',
                        war: 'target/devops-multi-env.war'
                    }
                }

                stage('Verify SIT') {
                    steps {
                        sh 'curl --fail --silent --show-error --retry 5 --retry-delay 2 "${TOMCAT_URL}/sit/" -o /dev/null'
                        echo 'SIT deployment verified successfully.'
                    }
                }
            }
        }

        // 3. This stage runs on the Jenkins built-in node WITHOUT consuming a heavy build executor slot
        stage('UAT Approval') {
            agent none 
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input(
                        message: 'DEV and SIT are deployed. Approve deployment to UAT?',
                        ok: 'Approve UAT'
                    )
                }
            }
        }

        // 4. If approved, request a new available agent to perform the final deployment steps
        stage('UAT Deployment') {
            agent any
            stages {
                stage('Deploy UAT') {
                    steps {
                        echo 'Deploying to UAT...'
                        deploy adapters: [
                            tomcat9(credentialsId: 'TomcatCreds', url: "${TOMCAT_URL}")
                        ],
                        contextPath: '/uat',
                        war: 'target/devops-multi-env.war'
                    }
                }

                stage('Verify UAT') {
                    steps {
                        sh 'curl --fail --silent --show-error --retry 5 --retry-delay 2 "${TOMCAT_URL}/uat/" -o /dev/null'
                        echo 'UAT deployment verified successfully.'
                    }
                }
            }
        }
    }

    post {
        success { echo 'All pipeline stages completed successfully.' }
        failure { echo 'Pipeline failed. Check the Jenkins Console Output.' }
        always  { echo 'Pipeline finished.' }
    }
}
