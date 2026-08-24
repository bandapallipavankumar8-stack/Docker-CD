pipeline {
    agent any

    environment {
        // Explicit S3 Bucket Name
        S3_BUCKET    = 'code-version'
        PACKAGE_NAME = 'bookstore-package.zip'
        AWS_CREDS_ID = 'aws-credentials-id'  // Matches your Jenkins AWS credentials store ID
        
        // Target Amazon Linux VM Configuration Details
        VM_IP        = '3.110.118.236'
        VM_USER      = 'ec2-user'      
        SSH_CREDS_ID = 'vm-ssh-key'    // Matches your Jenkins VM private key store ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling the deployment pipeline configuration...'
                checkout scm
            }
        }

        stage('Secure S3 Fetch and Deploy') {
            steps {
                // 1. Pull both your secure AWS keys and your SSH key from the Jenkins vault safely
                withCredentials([usernamePassword(credentialsId: "${AWS_CREDS_ID}", 
                                                 usernameVariable: 'AWS_ACCESS_KEY_ID', 
                                                 passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    
                    sshagent(credentials: ["${SSH_CREDS_ID}"]) {
                        echo "Connecting securely to Amazon Linux VM: ${VM_IP}..."
                        
                        sh """
                            ssh -o StrictHostKeyChecking=no ${VM_USER}@${VM_IP} '
                                echo "Successfully logged in! Refreshing system utilities..."
                                
                                # 2. Ensure Nginx, unzip, and AWS CLI are fully installed on the machine
                                sudo yum install -y nginx unzip awscli --skip-broken
                                
                                # 3. Make sure the Nginx web engine is turned on and active
                                sudo systemctl enable --now nginx
                                
                                # 4. Create an isolated workspace folder inside /tmp to bypass root blocks
                                cd /tmp
                                sudo rm -rf s3-secure-deploy
                                mkdir -p s3-secure-deploy
                                cd s3-secure-deploy
                                
                                # 5. Pass your Jenkins AWS keys into the shell environment memory
                                export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                                export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                                export AWS_DEFAULT_REGION=ap-south-1
                                
                                echo "Downloading your package securely via AWS CLI from bucket: ${S3_BUCKET}..."
                                aws s3 cp s3://${S3_BUCKET}/${PACKAGE_NAME} .
                                
                                # 6. Extract the zip file layers completely
                                echo "Extracting application assets..."
                                unzip -o ${PACKAGE_NAME}
                                
                                # 7. Handle nested sub-folders automatically if they exist
                                if [ ! -f "index.html" ]; then
                                    mv */index.html . 2>/dev/null || true
                                fi
                                
                                # 8. Wipe out old configs and copy your fresh code file over to the root path
                                sudo rm -rf /usr/share/nginx/html/*
                                sudo cp index.html /usr/share/nginx/html/
                                
                                # 9. Fix directory tree read and execute paths to permanently stop 403 blocks
                                sudo chmod 755 /usr /usr/share /usr/share/nginx /usr/share/nginx/html
                                sudo chmod 644 /usr/share/nginx/html/index.html
                                sudo chown -R nginx:nginx /usr/share/nginx/html
                                
                                # 10. Clear SELinux context rules for modern web hosting security policies
                                sudo chcon -Rt httpd_sys_content_t /usr/share/nginx/html 2>/dev/null || true
                                
                                # 11. Clean up temporary deployment paths from the server
                                cd /tmp
                                rm -rf s3-secure-deploy
                                
                                # 12. Restart Nginx to pick up your live bookstore landing page layout
                                sudo systemctl restart nginx
                                echo "Deployment script execution finished!"
                            '
                        """
                    }
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
