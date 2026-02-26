pipeline {
    agent any

    environment {
        IMAGE_NAME = "yourdockerhubusername/spring-petclinic"
        DOCKER_CREDS = credentials('dockerhub-creds')
        SONAR_TOKEN = credentials('sonar-token')
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
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
                        sh 'mvn test -Dtest=*ControllerTest'
                    }
                }
                stage('Service Tests') {
                    steps {
                        sh 'mvn test -Dtest=*ServiceTest'
                    }
                }
            }
        }

        stage('Static Scan - SonarQube') {
            steps {
                sh """
                mvn sonar:sonar \
                -Dsonar.projectKey=petclinic \
                -Dsonar.host.url=http://host.docker.internal:9000 \
                -Dsonar.login=${SONAR_TOKEN}
                """
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
