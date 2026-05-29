    pipeline {
        agent {
            docker {
                image 'node:18'
                args '-v /var/run/docker.sock:/var/run/docker.sock -u root'
            }
        }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage ('Install Docker') {
            steps {
                sh 'apt-get update && apt-get install -y docker.io'
                sh 'apt-get install wget gnupg -y'
                sh 'wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor |  tee /usr/share/keyrings/trivy.gpg > /dev/null'
                sh 'echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | tee -a /etc/apt/sources.list.d/trivy.list'
                sh 'apt-get update'
                sh 'apt-get install trivy -y'
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
            sh "docker run --rm -i hadolint/hadolint < Dockerfile"
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
