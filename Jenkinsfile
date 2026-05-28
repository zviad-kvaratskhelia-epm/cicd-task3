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
                def imageName = BRANCH_NAME == "main" ? "d3f4ault/nodemain:v1.0" : "d3f4ault/nodedev:v1.0"
                sh "docker build -t ${imageName} ."
            }
        }
    }
        stage("Push") {
            steps {
                script {
                def imageName = BRANCH_NAME == "main" ? "d3f4ault/nodemain:v1.0" : "d3f4ault/nodedev:v1.0"
                docker.withRegistry("https://registry.hub.docker.com", "c56850e3-4517-420b-b6e1-1f7d78eee9ba") {
                withCredentials([usernamePassword(
                    credentialsId: 'c56850e3-4517-420b-b6e1-1f7d78eee9ba',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                        sh "docker login -u $USER -p $PASS"
                        sh "docker push ${imageName}"
}
                sh "docker push ${imageName}"
            }
        }
    }
}
        stage('Trigger Deploy') {
            steps {
                script {
                    if(BRANCH_NAME == "main") {
                        build job: "Deploy_to_main"
                    }
                    else {
                        build job: "Deploy_to_dev"
                    }
                }
            }
        }
    }
}
