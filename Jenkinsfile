
pipeline {
    agent any

    tools {
        terraform 'terraform 1.9.8' // Ensure this version is configured in Jenkins
    }

    environment {
        GIT_REPO_URL = 'github.com/Akhil0907/test.git'
        GIT_BRANCH = 'master'
        GIT_CREDENTIALS_ID = 'auth_token' // Replace with your actual credentials ID
        MFA_PROFILE = 'akhil-mfa' // Replace with your actual AWS profile
        AWS_REGION = 'us-east-1'
        AWS_CREDENTIALS_ID = 'aws-credentials' // Replace with your actual credentials ID
        AWS_CREDENTIALS_FILE = 'aws_credentials.json' // Path to the file in the Jenkins workspace
        PYTHON_VERSION = '3.8.10' // Specify the Python version to install
        PYTHON_INSTALL_DIR = "${env.WORKSPACE}/python" // Custom installation directory for Python
        PATH = "${env.PYTHON_INSTALL_DIR}/bin:${env.PATH}" // Add Python to PATH
    }

    stages {
        stage('Checkout') {
            steps {
                withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                    git clone --branch $GIT_BRANCH https://$GIT_USERNAME:$GIT_PASSWORD@$GIT_REPO_URL
                    '''
                }
            }
        }
        
       stage('Terraform Init') {
            steps {
                script {
                    sh 'terraform init'
                }
            }
        }

        
        stage('Install AWS CLI') {
            steps {
                // Install AWS CLI
                sh '''
                if ! command -v aws &> /dev/null
                then
                    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                    unzip awscliv2.zip
                    ./aws/install -b ~/bin/aws
                fi
                '''
            }
        }
       
    }
       /* stage('Restore DynamoDB Table') {
            steps {
                // Restore the DynamoDB table to a specific point in time
                withEnv([
                    "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                    "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                    "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
                ]) {
                    sh '''
                    aws dynamodb restore-table-to-point-in-time \
                    --source-table-name sandbox-poc-akhil-bkp-1 \
                    --target-table-name sandbox-poc-akhil-bkp-2 \
                    --no-use-latest-restorable-time \
                    --restore-date-time 729755290.56 \
                    --region $AWS_REGION
                    '''
                }
            }
        }

        stage('Wait for table to be created') {
            steps {
                // Wait for the restored table to exist
                withEnv([
                    "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                    "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                    "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
                ]) {
                    sh '''
                    aws dynamodb wait table-exists \
                    --table-name 'sandbox-poc-akhil-bkp-1' \
                    --region $AWS_REGION
                    '''
                }
            }
        }

        stage('Terraform State Management') {
            steps {
                // Remove all resources from the Terraform state
                script {
                    withEnv([
                        "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                        "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                        "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
                    ]) {
                        def resources = sh(script: 'terraform state list', returnStdout: true).trim().split('\n')
                        for (resource in resources) {
                            sh "terraform state rm ${resource}"
                        }
                    }
                }
            }
        }

        stage('Terraform Import') {
            steps {
                // Import the existing DynamoDB table
                withEnv([
                    "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                    "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                    "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
                ]) {
                    sh 'terraform import aws_dynamodb_table.content backup_table_4'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                // Plan the Terraform changes
                withEnv([
                    "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                    "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                    "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
                ]) {
                    script {
                        def output = sh(script: 'terraform output -json', returnStdout: true).trim()
                        def json = readJSON text: output
                        env.TABLE_NAME = json.dynamodb_table_name.value
                    }
                    sh "terraform plan -var='table_name=${env.TABLE_NAME}'"
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                // Apply the Terraform changes
                withEnv([
                    "AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}",
                    "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}",
                    "AWS_SESSION_TOKEN=${env.AWS_SESSION_TOKEN}"
                ]) {
                    sh "terraform apply -var='table_name=${env.TABLE_NAME}' -auto-approve"
                }
            }
        }
    }*/

    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
