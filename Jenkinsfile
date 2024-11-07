pipeline {
    agent any

    tools {
        terraform 'terraform 1.9.8' // Ensure this version is configured in Jenkins
    }

    environment {
        AWS_REGION = 'us-east-1'
        AWS_CREDENTIALS_ID = 'aws_credentials' // Replace with your actual credentials ID
        AWS_CREDENTIALS_FILE = 'aws_credentials.json' // Path to the file in the Jenkins workspace
        AWS_CLI_DIR = "${env.WORKSPACE}/aws-cli" // Custom installation directory for AWS CLI
        CREDENTIALS_ID = 'github_ssh_key'
        BRANCH_NAME = 'master'
        REPO_NAME = 'test'
        ENV_URL = "git@github.com:Akhil0907/${REPO_NAME}.git"
    }

    stages {
        stage('Checkout') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${CREDENTIALS_ID}", keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                    GIT_SSH_COMMAND="ssh -i $SSH_KEY -o StrictHostKeyChecking=no" git clone --depth=1 --branch ${BRANCH_NAME} ${ENV_URL}
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
                    ./aws/install -i ${AWS_CLI_DIR} -b ${AWS_CLI_DIR}/bin
                fi
                '''
            }
        }

        stage('List DynamoDB Tables') {
            steps {
                withCredentials([file(credentialsId: 'aws_credentials', variable: 'AWS_CREDENTIALS_FILE')]) {
                    script {
                        // Read AWS credentials from the JSON file using readJSON step
                        def awsCredentials = readJSON file: AWS_CREDENTIALS_FILE
                        def accessKeyId = awsCredentials.AccessKeyId
                        def secretAccessKey = awsCredentials.SecretAccessKey
                        def sessionToken = awsCredentials.SessionToken

                        withEnv([
                            "AWS_ACCESS_KEY_ID=${accessKeyId}",
                            "AWS_SECRET_ACCESS_KEY=${secretAccessKey}",
                            "AWS_SESSION_TOKEN=${sessionToken}"
                        ]) {
                            // List DynamoDB tables to verify AWS and Jenkins connection
                            sh '''
                            aws dynamodb list-tables --region $AWS_REGION
                            '''
                        }
                    }
                }
            }
        }
    }
        /*stage('Restore DynamoDB Table') {
            steps {
                script {
                    // Restore the DynamoDB table to a specific point in time
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
                script {
                    // Wait for the restored table to exist
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
                script {
                    // Remove all resources from the Terraform state
                    def resources = sh(script: 'terraform state list', returnStdout: true).trim().split('\n')
                    for (resource in resources) {
                        sh "terraform state rm ${resource}"
                    }
                }
            }
        }

        stage('Terraform Import') {
            steps {
                script {
                    // Import the existing DynamoDB table
                    sh 'terraform import aws_dynamodb_table.content backup_table_4'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    // Plan the Terraform changes
                    def output = sh(script: 'terraform output -json', returnStdout: true).trim()
                    def json = readJSON text: output
                    env.TABLE_NAME = json.dynamodb_table_name.value
                    sh "terraform plan -var='table_name=${env.TABLE_NAME}'"
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    // Apply the Terraform changes
                    sh "terraform apply -var='table_name=${env.TABLE_NAME}' -auto-approve"
                }
            }
        }
    }
*/
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
