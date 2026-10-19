```groovy
pipeline {
    agent any

    tools {
        maven 'Maven 3.9.9'
        jdk 'JDK21'
    }

    environment {
        APP_NAME = 'devops-multi-env'
        TOMCAT_URL = 'http://172.31.47.158:8090'
    }

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
                    tomcat9(
                        credentialsId: 'TomcatCreds',
                        url: "${TOMCAT_URL}"
                    )
                ],
                contextPath: '/dev',
                war: 'target/devops-multi-env.war'
            }
        }

        stage('Verify DEV') {
            steps {
                echo 'DEV deployment completed.'
            }
        }

        stage('Deploy SIT') {
            steps {
                echo 'Deploying to SIT...'
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'TomcatCreds',
                        url: "${TOMCAT_URL}"
                    )
                ],
                contextPath: '/sit',
                war: 'target/devops-multi-env.war'
            }
        }

        stage('Verify SIT') {
            steps {
                echo 'SIT deployment completed.'
            }
        }

        stage('UAT Approval') {
            steps {
                input message: 'Approve deployment to UAT?'
            }
        }

        stage('Deploy UAT') {
            steps {
                echo 'Deploying to UAT...'
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'TomcatCreds',
                        url: "${TOMCAT_URL}"
                    )
                ],
                contextPath: '/uat',
                war: 'target/devops-multi-env.war'
            }
        }

        stage('Verify UAT') {
            steps {
                echo 'UAT deployment completed.'
            }
        }
    }

    post {
        success {
            echo 'Multi-environment pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins Console Output.'
        }

        always {
            echo 'Pipeline finished.'
        }
    }
}
```
