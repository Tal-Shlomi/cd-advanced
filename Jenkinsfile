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
        stage('Pull Image from ECR on EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: '123',
                        keyFileVariable: 'SSH_KEY_FILE'
                    )
                ]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@10.0.1.177 '
                            aws ecr get-login-password --region us-east-1 | \
                            docker login --username AWS --password-stdin \
                            992382545251.dkr.ecr.us-east-1.amazonaws.com &&
                            docker pull 992382545251.dkr.ecr.us-east-1.amazonaws.com/test-flask-app:latest
                        '
                    '''
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: '<credentials_id>',
                        keyFileVariable: 'SSH_KEY_FILE'
                    )
                ]) {
                    sh 'scp -o StrictHostKeyChecking=no -i $SSH_KEY_FILE Dockerrun.aws.json ec2-user@<ec2_instance_ip>:/home/ec2-user/'
                    sh 'ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@<ec2_instance_ip> "docker stop <container_name> || true"'
                    sh 'ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@<ec2_instance_ip> "docker rm <container_name> || true"'
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY_FILE ec2-user@<ec2_instance_ip> \
                        "docker run -d --name <container_name> -p 80:8080 \
                        <aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/<image_name>:latest"
                    '''
                }
            }
        }
    }
}
