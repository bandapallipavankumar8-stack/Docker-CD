pipeline {
    agent any

    environment {
        // Target Amazon Linux VM Configuration Details
        VM_IP        = '3.110.118.236'
        VM_USER      = 'ec2-user'      // Default administrative user account for Amazon Linux
        SSH_CREDS_ID = 'vm-ssh-key'    // The ID of the private key file saved in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling the deployment pipeline configuration...'
                checkout scm
            }
        }

        stage('Install Nginx and Deploy on Amazon Linux VM') {
            steps {
                // Uses the Jenkins SSH agent plugin to inject your private key file securely
                sshagent(credentials: ["${SSH_CREDS_ID}"]) {
                    echo "Establishing secure admin connection to Amazon Linux VM: ${VM_IP}..."
                    
                    sh """
                        ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_IP} '
                            echo "Successfully authenticated! Installing native web engine..."
                            
                            # 1. Clean package metadata and update repository references
                            sudo dnf clean all
                            sudo dnf makecache
                            
                            # 2. Install native Nginx web server, curl, and unzip tools
                            sudo dnf install -y nginx curl unzip
                            
                            # 3. Enable Nginx and start the background service
                            sudo systemctl enable nginx
                            sudo systemctl start nginx
                            
                            # 4. Clean out any default placeholder greeting pages
                            sudo rm -rf /usr/share/nginx/html/*
                            
                            # 5. Download your live storefront page package from GitHub
                            curl -L -o repo.zip https://github.com
                            
                            # 6. Extract the zipped repository archive
                            unzip -o repo.zip
                            
                            # 7. Copy your bookstore index.html file straight into the live Nginx web directory
                            sudo cp Docker-CI-main/index.html /usr/share/nginx/html/
                            
                            # 8. Purge temporary setup files to keep the server space clean
                            rm -rf repo.zip Docker-CI-main
                            
                            # 9. Force-restart the daemon to apply your custom webpage layout
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
