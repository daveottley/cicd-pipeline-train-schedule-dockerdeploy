pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Deploy to Docker') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("daveottley/train-schedule")
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
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([sshUserPrivateKey(credentialsId: 'webserver_deploy_key', usernameVariable: 'USERNAME')]) {
                    script {
                        sh "ssh -o StrictHostKeyChecking=no ${env.USERNAME}@${env.prod_ip} \"docker pull daveottley/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -o StrictHostKeyChecking=no ${env.USERNAME}@${env.prod_ip} \"docker stop train-schedule\""
                            sh "ssh -o StrictHostKeyChecking=no ${env.USERNAME}@${env.prod_ip} \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -o StrictHostKeyChecking=no ${env.USERNAME}@${env.prod_ip} \"docker run --restart always --name train-schedule -p 80:8080 -d daveottley/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
