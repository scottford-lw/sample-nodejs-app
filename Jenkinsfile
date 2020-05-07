pipeline {
    agent { label any }
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
            when {
                branch 'master'
            }
            steps {
                echo 'Running Lacework vulnerability scan'
                sh 'lacework vulnerability vul report sha256:fa31faba325ec777ad2a5d72689c4f9e1e31bc1e9bc3bd2fd98197e0d1125688 --details'
            }
        }
    }
}
