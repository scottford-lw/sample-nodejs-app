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
                    app = docker.build("selacework/sample-nodejs-app")
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
                    docker.withRegistry('https://registry.hub.docker.com', 'selacework_docker_hub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Lacework Vulnerability Scan') {
            environment {
                LW_ACCOUNT = credentials('LW_ACCOUNT')
                LW_API_KEY = credentials('LW_API_KEY')
                LW_API_SECRET = credentials('LW_API_SECRET')
            }
            when {
                branch 'master'
            }
            steps {
                echo 'Running Lacework vulnerability scan'
                sh 'lacework vulnerability container scan index.docker.io selacework/sample-nodejs-app latest --poll --noninteractive --html --account $LW_ACCOUNT --api_key $LW_API_KEY --api_secret $LW_API_SECRET'
            }
        }
    }
}
