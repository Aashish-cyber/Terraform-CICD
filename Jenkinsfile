pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('60b5451a-c216-4ab3-9914-e91c3e90d8ca')  // Replace with your AWS credentials ID
        AWS_SECRET_ACCESS_KEY = credentials('60b5451a-c216-4ab3-9914-e91c3e90d8ca')  // Replace with your AWS credentials ID
        EC2_KEY_CREDENTIAL_ID = 'a8044eac-fbb2-4b72-bfe1-3055d4414995'  // Replace with your Jenkins credential ID for EC2 private key
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/Aashish-cyber/Terraform-CICD.git'  // Replace with your GitHub repository URL
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -auto-approve'
            }
        }

        stage('Deploy Flask App') {
            steps {
                sshagent([a8044eac-fbb2-4b72-bfe1-3055d4414995]) {
                    script {
                        def ec2PublicIp = sh(script: 'terraform output -raw 65.2.131.64', returnStdout: true).trim()

                        // Copy the app to the EC2 instance
                        sh "scp -o StrictHostKeyChecking=no app.py ubuntu@${65.2.131.64}:/home/ubuntu/"

                        // SSH into the EC2 instance and deploy the app
                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${65.2.131.64} << EOF
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
