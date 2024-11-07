// Declare variables as Groovy string variables
String awsCredentialsId = 'aws_credentials' // Replace with your actual credentials ID
String awsCredentialsFile = 'aws_credentials.json' // Path to the file in the Jenkins workspace
String credentialsId = 'github_ssh_key'
String branchName = 'master'
String repoName = 'test'
String envUrl = "git@github.com:Akhil0907/${repoName}.git"

pipeline {
    agent any

    tools {
        terraform 'terraform 1.9.8' // Ensure this version is configured in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: credentialsId, keyFileVariable: 'SSH_KEY')]) {
                    sh """
                    GIT_SSH_COMMAND="ssh -i \$SSH_KEY -o StrictHostKeyChecking=no" git clone --depth=1 --branch ${branchName} ${envUrl}
                    """
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
             script {
                 // Set the AWS CLI directory
                 def awsCliDir = "${env.WORKSPACE}/aws-cli"

                // Install AWS CLI
                 sh """
                if ! command -v aws &> /dev/null
               then
                curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                unzip awscliv2.zip
                ./aws/install -i ${awsCliDir} -b ${awsCliDir}/bin
            fi
            """
        }
    }
}
        
     /* stage('Read AWS Credentials') {
            steps {
                withCredentials([file(credentialsId: 'aws_credentials', variable: 'AWS_CREDENTIALS_FILE')]) {
                    script {
                        // Read AWS credentials from the JSON string
                        def awsCredentials = readJSON file: AWS_CREDENTIALS_FILE
                        env.AWS_ACCESS_KEY_ID = awsCredentials.AccessKeyId
                        env.AWS_SECRET_ACCESS_KEY = awsCredentials.SecretAccessKey
                        env.AWS_SESSION_TOKEN = awsCredentials.SessionToken
                    }
                }
            }
        }*/

       stage('Read AWS Credentials') {
            steps {
                withCredentials([file(credentialsId: awsCredentialsId, variable: 'AWS_CREDENTIALS_FILE')]) {
                    script {
                        // Read AWS credentials from the JSON file using readJSON step
                        def awsCredentials = readJSON file: AWS_CREDENTIALS_FILE
                        env.AWS_ACCESS_KEY_ID = awsCredentials.AccessKeyId
                        env.AWS_SECRET_ACCESS_KEY = awsCredentials.SecretAccessKey
                        env.AWS_SESSION_TOKEN = awsCredentials.SessionToken
                    }
                }
            }
        }

        stage('List DynamoDB Tables') {
            steps {
                withAWS(region: awsRegion, credentials: awsCredentialsId) {
                    sh '''
                    aws dynamodb list-tables --region $AWS_REGION
                    '''
                }
            }
        }
    }
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}
