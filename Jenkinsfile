pipeline {
    agent any

    environment {
        // Target Amazon Linux VM Configuration Details
        VM_IP        = '3.110.118.236'
        VM_USER      = 'ec2-user'      // Correct default user account profile for Amazon Linux
        SSH_CREDS_ID = 'vm-ssh-key'    // The ID of the private key file saved in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling the latest code files from your GitHub repository...'
                checkout scm
            }
        }

        stage('Install and Deploy on Amazon Linux VM') {
            steps {
                // Uses the Jenkins SSH agent plugin to inject your private key file securely
                sshagent(credentials: ["${SSH_CREDS_ID}"]) {
                    echo "Establishing secure admin connection to Amazon Linux VM: ${VM_IP}..."
                    
                    sh """
                        ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_IP} '
                            echo "Successfully authenticated! Starting Red Hat environment configuration..."
                            
                            # 1. Update the server package manager repository cache
                            sudo yum update -y
                            
                            # 2. Install the native Nginx web server engine, curl, and unzip utilities
                            sudo yum install nginx curl unzip -y
                            
                            # 3. Ensure Nginx service is enabled and starts automatically if the instance reboots
                            sudo systemctl enable --now nginx
                            
                            # 4. Clean out any default Amazon Linux Nginx landing page placeholder files
                            sudo rm -rf /usr/share/nginx/html/*
                            
                            # 5. Pull your live code branch bundle package directly from GitHub
                            curl -L -o repo.zip https://github.com
                            
                            # 6. Extract the zipped repository archive layer folders
                            unzip -o repo.zip
                            
                            # 7. Copy your bookstore index.html file straight into the live Amazon Nginx web directory
                            sudo cp Docker-CI-main/index.html /usr/share/nginx/html/
                            
                            # 8. Purge temporary setup files from the directory tree to keep the server clean
                            rm -rf repo.zip Docker-CI-main
                            
                            # 9. Restart the system Nginx daemon tool to refresh and render your bookstore layout
                            sudo systemctl restart nginx
                            
                            echo "Success! Your bookstore web app is live on Amazon Linux."
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline Finished Successfully! Your application is live at http://3.110.118.236'
        }
        failure {
            echo 'Deployment Failed. Please check the console log outputs to troubleshoot the error.'
        }
    }
}
