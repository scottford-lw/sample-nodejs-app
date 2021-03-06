pipeline {
    agent { label 'ubuntu1804_runner' }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("techallylw/sample-nodejs-app")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_techallylw') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Lacework Vulnerability Scan') {
            environment {
                LW_API_SECRET = credentials('lacework_api_secret')
            }
            when {
                branch 'master'
            }
            steps {
                echo 'Running Lacework vulnerability scan'
                sh 'lacework vuln ctr scan index.docker.io techallylw/sample-nodejs-app latest --poll --noninteractive --details'
            }
        }
    }
}
