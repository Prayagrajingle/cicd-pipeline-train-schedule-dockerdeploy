pipeline {
    agent any

    environment {
        DOCKER_HUB_USERNAME = 'your_dockerhub_username' // Replace with your actual Docker Hub username
        prod_ip = 'your.production.server.ip'           // Replace with your production server IP
    }

    stages {
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        def dockerImage = "${DOCKER_HUB_USERNAME}/train-schedule:${env.BUILD_NUMBER}"
                        def sshCmd = { cmd ->
                            return "sshpass -p '$USERPASS' ssh -o StrictHostKeyChecking=no $USERNAME@${prod_ip} \"${cmd}\""
                        }

                        // Pull the latest image
                        sh sshCmd("docker pull ${dockerImage}")

                        // Stop and remove existing container
                        try {
                            sh sshCmd("docker stop train-schedule")
                            sh sshCmd("docker rm train-schedule")
                        } catch (err) {
                            echo "Caught error during stop/remove: ${err}"
                        }

                        // Run the new container
                        sh sshCmd("docker run --restart always --name train-schedule -p 8080:8080 -d ${dockerImage}")
                    }
                }
            }
        }
    }
}
