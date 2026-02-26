pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9'
    }
    
    environment {
        IMAGE_NAME = "akhilanelapatla111/spring-petclinic"
        DOCKER_CREDS = credentials('dockerhub-creds')
        SONAR_TOKEN = credentials('sonar-token')
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/akhilanelapatla111/spring-petclinic.git'
                    ]]
                ])
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Tests') {
            parallel {
                stage('Controller Tests') {
                    steps {
                        sh 'mvn test -Dtest=*ControllerTests'
                    }
                }
                stage('Service Tests') {
                    steps {
                        sh 'mvn test -Dtest=*ServiceTests'
                    }
                }
            }
        }

        stage('Static Scan - SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw clean verify sonar:sonar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh """
                echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin
                docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                """
            }
        }
    }

    post {
        always {
            script {
                echo "Total Pipeline Duration: ${currentBuild.durationString}"
            }
        }
    }
}
