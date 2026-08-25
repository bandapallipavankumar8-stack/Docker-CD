pipeline {
    agent any

    environment {
        // AWS Settings
        AWS_BUCKET       = 'code-version'
        ZIP_NAME         = 'bookstore-package.zip'
        
        // YOUR UPDATED JENKINS CREDENTIAL ID
        AWS_CRED_ID      = 'aws-credentials-id' 
        
        // Amazon Linux EC2 Settings
        EC2_USER         = 'ec2-user'        
        EC2_IP           = '43.204.219.68'   
        
        // PLEASE VERIFY THIS SSH CREDENTIAL ID MATCHES JENKINS
        SSH_CRED_ID      = 'ec2-ssh-key'     
        
        // Docker Settings
        IMAGE_NAME       = 'bookstore-app'
        CONTAINER_NAME   = 'bookstore-container'
    }

    stages {
        stage('CD: Checkout Config') {
            steps {
                echo 'Pulling the deployment pipeline configuration...'
            }
        }

        stage('CD: Fetch and Deploy Docker Container') {
            steps {
                // Securely pulls your Username (Access Key) and Password (Secret Key)
                withCredentials([usernamePassword(credentialsId: "${AWS_CRED_ID}", passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    echo "Downloading package from S3..."
                    sh "aws s3 cp s3://${AWS_BUCKET}/${ZIP_NAME} ."
                }

                // Deploy package over SSH using your Jenkins SSH key
                sshagent([SSH_CRED_ID]) {
                    echo "Deploying to Amazon Linux at ${EC2_IP}..."
                    
                    // Copy file to EC2 home directory
                    sh "scp -o StrictHostKeyChecking=no ${ZIP_NAME} ${EC2_USER}@${EC2_IP}:/home/${EC2_USER}/"
                    
                    // Run deployment steps inside target instance
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                            mkdir -p /home/${EC2_USER}/app
                            rm -rf /home/${EC2_USER}/app/*
                            
                            # Extract package content
                            unzip -o /home/${EC2_USER}/${ZIP_NAME} -d /home/${EC2_USER}/app/
                            rm -f /home/${EC2_USER}/${ZIP_NAME}
                            
                            cd /home/${EC2_USER}/app
                            
                            # Re-create Docker container
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                            docker build -t ${IMAGE_NAME}:latest .
                            docker run -d -p 80:80 --name ${CONTAINER_NAME} ${IMAGE_NAME}:latest
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Jenkins workspace...'
            cleanWs()
        }
    }
}
