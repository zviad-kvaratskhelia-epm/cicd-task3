pipeline {
    agent {
        docker {
            image 'node:16'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u root'
        }
    }
}
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
        stage('Lint docker file') {
            steps {
                sh "hadolint Dockerfile"
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
        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
            def imageName = BRANCH_NAME == "main" ? "d3f4ault/nodemain:v1.0" : "d3f4ault/nodedev:v1.0"
            def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${imageName}", returnStdout: true).trim()
            echo "Vulnerability Report:\n${vulnerabilities}"
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
                        build job: "Deploy_to_main", wait: false
                    }
                    else {
                        build job: "Deploy_to_dev", wait: false
                    }
                }
            }
        }
    }
}
