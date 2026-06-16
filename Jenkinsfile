pipeline {
    agent any

    environment {
        // Updated with your Nginx instance's Public IP address
        NGINX_IP   = '65.2.168.166'
        
        // Target Nginx server user and directory
        NGINX_USER = 'ubuntu' 
        TARGET_DIR = '/var/www/html/'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "Fetching the latest template from GitHub..."
                // Targeted 'master' branch based on your repository setup
                git branch: 'master', url: 'https://github.com/shashankJ17/static-wed-hosting-via-jenkins.git'
            }
        }

        stage('Deploy to Nginx') {
            steps {
                // Uses the SSH Agent plugin with the credential ID you created in Jenkins
                sshagent(credentials: ['nginx-server-ssh']) {
                    echo "Adding target host to known_hosts to prevent SSH hanging..."
                    sh "mkdir -p ~/.ssh && chmod 700 ~/.ssh"
                    sh "ssh-keyscan -H ${NGINX_IP} >> ~/.ssh/known_hosts"
                    
                    echo "Syncing static web files to Nginx web root..."
                    // rsync pushes the workspace content to the Nginx server.
                    // --exclude removes git files so you aren't hosting your repository's metadata.
                    sh "rsync -avz --delete --exclude='.git*' ./ ${NGINX_USER}@${NGINX_IP}:${TARGET_DIR}"
                }
            }
        }
        
        stage('Verify & Reload Nginx') {
            steps {
                sshagent(credentials: ['nginx-server-ssh']) {
                    echo "Testing Nginx configuration and reloading service..."
                    // Verifies syntax first, then reloads gracefully without dropping connection traffic
                    sh "ssh ${NGINX_USER}@${NGINX_IP} 'sudo nginx -t && sudo systemctl reload nginx'"
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Deployment successful! Your static site is live.'
        }
        failure {
            echo '❌ Deployment failed. Please check the console output details.'
        }
    }
}
