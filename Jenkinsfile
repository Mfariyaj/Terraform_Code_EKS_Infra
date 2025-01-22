properties([
    parameters([
        choice(
            choices: ['dev', 'staging', 'prod'], 
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action'
        )
    ])
])

pipeline {
    agent any
    environment {
        TERRAFORM_DIR = 'eks/' // Set the Terraform directory as a variable
    }
    stages {
        stage('Preparing') {
            steps {
                echo 'Preparing for Terraform execution...'
            }
        }
        stage('Git Pulling') {
            steps {
                echo 'Pulling latest code from GitHub...'
                git branch: 'master', url: 'https://github.com/Mfariyaj/Terraform_Code_EKS_Infra.git'
            }
        }
        stage('Init') {
            steps {
                echo 'Initializing Terraform...'
                withAWS(credentials: 'aws_creds', region: 'us-east-1') {
                    sh "terraform -chdir=${TERRAFORM_DIR} init"
            }
        }
        }
        stage('Validate') {
            steps {
                echo 'Validating Terraform configuration...'
                withAWS(credentials: 'aws_creds', region: 'us-east-1') {
                    sh "terraform -chdir=${TERRAFORM_DIR} validate"
                }
            }
        }
        stage('Action') {
            steps {
                echo "Executing Terraform action: ${params.Terraform_Action}..."
                withAWS(credentials: 'aws_creds', region: 'us-east-1') {
                    script {    
                        switch (params.Terraform_Action) {
                            case 'plan':
                                sh "terraform -chdir=${TERRAFORM_DIR} plan -var-file=${params.Environment}.tfvars"
                                break
                            case 'apply':
                                sh "terraform -chdir=${TERRAFORM_DIR} apply -var-file=${params.Environment}.tfvars -auto-approve"
                                break
                            case 'destroy':
                                sh "terraform -chdir=${TERRAFORM_DIR} destroy -var-file=${params.Environment}.tfvars -auto-approve"
                                break
                            default:
                                error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                        }
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Terraform pipeline executed successfully!'
        }
        failure {
            echo 'Terraform pipeline failed. Check logs for details.'
        }
    }
}
