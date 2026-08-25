pipeline {
    agent any

    environment {
        // AWS Settings
        AWS_BUCKET       = 'code-version'
        ZIP_NAME         = 'bookstore-package.zip'
        AWS_CRED_ID      = 'aws-credentials' // Jenkins AWS Credential ID (if used)
        
        // Amazon Linux EC2 Settings
        EC2_USER         = 'ec2-user'        
        EC2_IP           = '43.204.219.68'   
        SSH_CRED_ID      = 'ec2-ssh-key'     // Jenkins SSH Private Key Credential ID
        
        // Docker App Settings
        IMAGE_NAME       = 'bookstore-app'
        CONTAINER_NAME   = 'bookstore-container'
    }

    stages {
        stage('CD: Fetch from S3') {
            steps {
                echo "Downloading ${ZIP_NAME} from S3 bucket..."
                // Downloads the specific zip file created by your CI step
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: "${AWS_CRED_ID}")]) {
                    sh "aws s3 cp s3://${AWS_BUCKET}/${ZIP_NAME} ."
                }
            }
        }

        stage('CD: Deploy to Amazon Linux EC2') {
            steps {
                // Securely injects your admin-configured SSH private key
                sshagent([SSH_CRED_ID]) {
                    echo "Transferring package to EC2: ${EC2_IP}..."
                    
                    // 1. Send the zip file securely to the target EC2 machine
                    sh "scp -o StrictHostKeyChecking=no ${ZIP_NAME} ${EC2_USER}@${EC2_IP}:/home/${EC2_USER}/"

                    echo "Unpacking and running application inside Docker container..."
                    // 2. SSH into EC2 to wipe the old app folder, unzip the new one, and re-build the container
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_IP} '
                            # Create and reset target workspace
                            mkdir -p /home/${EC2_USER}/app
                            rm -rf /home/${EC2_USER}/app/*
                            
                            # Extract files into workspace
                            unzip -o /home/${EC2_USER}/${ZIP_NAME} -d /home/${EC2_USER}/app/
                            rm -f /home/${EC2_USER}/${ZIP_NAME}
                            
                            cd /home/${EC2_USER}/app
                            
                            # Clean up old container version if it exists
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                            
                            # Build image using your updated Dockerfile
                            docker build -t ${IMAGE_NAME}:latest .
                            
                            # Fire up the active application container
                            docker run -d -p 80:80 --name ${CONTAINER_NAME} ${IMAGE_NAME}:latest
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Jenkins CD workspace...'
            cleanWs()
        }
    }
}
