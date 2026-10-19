pipeline {
    agent any

    tools {
        maven 'Maven 3.9.9'
        jdk 'JDK21' // Asegúrate de que este nombre coincida exactamente con el configurado en "Global Tool Configuration" de Jenkins
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

        stage('Deploy UAT') {
            steps {
                echo 'Deploying to UAT...'
                // Aquí puedes replicar la lógica de despliegue para UAT si lo deseas
            }
        }
    }
}
