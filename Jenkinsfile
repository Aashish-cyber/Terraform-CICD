pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('60b5451a-c216-4ab3-9914-e91c3e90d8ca')  // Replace with your AWS credentials ID
        AWS_SECRET_ACCESS_KEY = credentials('60b5451a-c216-4ab3-9914-e91c3e90d8ca')  // Replace with your AWS credentials ID
        EC2_KEY_CREDENTIAL_ID = 'a8044eac-fbb2-4b72-bfe1-3055d4414995'  // Replace with your Jenkins credential ID for EC2 private key
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Aashish-cyber/Terraform-CICD.git'  // Your GitHub repository
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform apply'
            }
        }

        stage('Retrieve EC2 Public IP') {
            steps {
                script {
                    // Retrieve the EC2 instance public IP directly from the state file
                    def ec2PublicIp = sh(script: 'terraform state show aws_instance.server | grep public_ip | awk \'{print $2}\'', returnStdout: true).trim()
                    
                    echo "EC2 Public IP: ${ec2PublicIp}"
                    
                    // Store the EC2 public IP in an environment variable for use in subsequent steps
                    env.EC2_PUBLIC_IP = ec2PublicIp
                }
            }
        }

        stage('Deploy Flask App') {
            steps {
                sshagent([EC2_KEY_CREDENTIAL_ID]) {
                    script {
                        // Copy the app to the EC2 instance
                        sh "scp -o StrictHostKeyChecking=no app.py ubuntu@${env.EC2_PUBLIC_IP}:/home/ubuntu/"

                        // SSH into the EC2 instance and deploy the app
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${env.EC2_PUBLIC_IP} << EOF
                        cd /home/ubuntu
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install flask
                        nohup python3 app.py &
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

