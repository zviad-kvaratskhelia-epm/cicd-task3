pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh "npm test"
            }
        }
        stage("Build Docker Image") {
            steps {
                script {
                def imageName = BRANCH_NAME == "main" ? "nodemain:v1.0" : "nodedev:v1.0"
                sh "docker build -t ${imageName} ."
            }
        }
    }
        stage('Deploy') {
            steps {
                script {
                    if(BRANCH_NAME == "main") {
                        sh '''
                            docker stop nodemain || true
                            docker rm nodemain || true
                            docker run -d --name nodemain -p 3000:3000 nodemain:v1.0
                           '''
                    }
                    else {
                        sh '''
                            docker stop nodedev || true
                            docker rm nodedev || true
                            docker run -d --name nodedev -p 3001:3000 nodedev:v1.0
                           '''
                    }
                }
            }
        }
    }
}