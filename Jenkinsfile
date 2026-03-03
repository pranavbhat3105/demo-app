pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')    // Jenkins credentials ID
        DOCKER_IMAGE          = 'prb3105/demo-app' // <-- CHANGE THIS
        SONAR_TOKEN           = credentials('sonar-token')         // Jenkins credentials ID
        SONAR_URL             = 'http://sonarqube:9000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    mvn sonar:sonar \
                      -Dsonar.host.url=${SONAR_URL} \
                      -Dsonar.login=${SONAR_TOKEN} \
                      -Dsonar.projectKey=demo-app
                """
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${DOCKER_IMAGE}:${tag} ."
                    sh "docker tag ${DOCKER_IMAGE}:${tag} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    def tag = "${env.BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE}:${tag}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
