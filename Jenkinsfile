pipeline {
    agent any

    environment {
        // Direct AWS S3 Storage Artifact Endpoint URL
        S3_PACKAGE_URL = 'https://amazonaws.com'
        PACKAGE_NAME   = 'bookstore-package.zip'
        
        // Target Amazon Linux VM Configuration Details
        VM_IP          = '3.110.118.236'
        VM_USER        = 'ec2-user'      // Default administrator profile for Amazon Linux
        SSH_CREDS_ID   = 'vm-ssh-key'    // The ID of the private key file saved in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling the deployment pipeline configuration...'
                checkout scm
            }
        }

        stage('Fetch S3 Asset and Deploy to VM') {
            steps {
                // Uses the Jenkins SSH agent plugin to inject your private key file securely
                sshagent(credentials: ["${SSH_CREDS_ID}"]) {
                    echo "Establishing secure admin connection to Amazon Linux VM: ${VM_IP}..."
                    
                    sh """
                        ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_IP} '
                            echo "Successfully authenticated! Refreshing package manager caches..."
                            
                            # 1. Force refresh yum configuration caches
                            sudo yum clean all
                            sudo yum makecache
                            
                            # 2. Ensure Nginx, curl, and unzip tools are completely installed on the server
                            sudo yum install -y nginx curl unzip --skip-broken
                            
                            # 3. Verify Nginx system daemon service is enabled and actively running
                            sudo systemctl enable nginx
                            sudo systemctl start nginx
                            
                            # 4. Clean out any default Amazon Linux placeholder greeting files
                            sudo rm -rf /usr/share/nginx/html/*
                            
                            # 5. Create a clean isolated temporary environment folder path
                            mkdir -p ~/s3-package-deploy
                            cd ~/s3-package-deploy
                            rm -f ${PACKAGE_NAME} index.html Dockerfile
                            
                            # 6. Securely stream and download the file directly from your explicit S3 asset bucket link
                            echo "Streaming artifact bundle from S3 storage layer..."
                            curl -L -o ${PACKAGE_NAME} ${S3_PACKAGE_URL}
                            
                            # 7. Unpack your archived code package layers 
                            unzip -o ${PACKAGE_NAME}
                            
                            # 8. FIX POTENTIAL NESTED SUB-FOLDERS: If the zip created a subfolder, bring index.html out to the root
                            if [ ! -f "index.html" ]; then
                                echo "index.html not found in root, looking in subdirectories..."
                                mv */index.html . 2>/dev/null || true
                            fi
                            
                            # 9. Copy your custom bookstore index.html file straight into the live Nginx web directory
                            sudo cp index.html /usr/share/nginx/html/
                            
                            # 10. CRITICAL ANTI-403 PERMISSION FIXES: Grant Nginx access to the folder path and files
                            echo "Applying permissions and ownership rules to prevent 403 Forbidden errors..."
                            sudo chmod 755 /usr/share/nginx /usr/share/nginx/html
                            sudo chown -R nginx:nginx /usr/share/nginx/html/* 2>/dev/null || true
                            sudo chmod 644 /usr/share/nginx/html/index.html 2>/dev/null || true
                            
                            # 11. Clean up the temporary workspace directories from the user system folder tree
                            cd ~
                            rm -rf ~/s3-package-deploy
                            
                            # 12. Restart the native Nginx tool service engine to hard-refresh your bookshop interface page
                            sudo systemctl restart nginx
                            
                            echo "Success! Your bookstore package has been retrieved from S3 and deployed live to Port 80."
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
