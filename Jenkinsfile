pipeline {
    agent any

    stages {

        stage('Checkout Repository') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Tal-Shlomi/cd-advanced.git'
                    ]]
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t test-flask-app .'
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 992382545251.dkr.ecr.us-east-1.amazonaws.com/test-flask-app'
                    sh 'docker push 992382545251.dkr.ecr.us-east-1.amazonaws.com/test-flask-app:latest'
            }
        }

        stage('Deploy Docker Container') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: '123',   
                        keyFileVariable: 'SSH_KEY_FILE'
                    )
                ]) {
                    sh '''
                        # Copy Dockerrun.aws.json (optional)
                        scp -o StrictHostKeyChecking=no -i $SSH_KEY_FILE Dockerrun.aws.json \
                        ec2-user@10.0.1.177:/home/ec2-user/ || true
            
                        # Stop and remove existing container (if any)
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@10.0.1.177 \
                        "docker stop test-flask-app || true && docker rm test-flask-app || true"
            
                        # Log into ECR and pull the latest image
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@10.0.1.177 '
                            aws ecr get-login-password --region us-east-1 | \
                            docker login --username AWS --password-stdin 992382545251.dkr.ecr.us-east-1.amazonaws.com && \
                            docker pull 992382545251.dkr.ecr.us-east-1.amazonaws.com/test-flask-app:latest
                        '
            
                        # Run the new container
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@10.0.1.177 \
                        "docker run -d --name test-flask-app -p 80:8080 \
                        992382545251.dkr.ecr.us-east-1.amazonaws.com/test-flask-app:latest"
                    '''
                }
            }

        }
    }
}
