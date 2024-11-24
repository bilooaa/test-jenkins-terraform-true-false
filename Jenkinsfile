pipeline {
    agent any

    parameters {
        booleanParam(name: 'CREATE_RESOURCES', defaultValue: true, description: 'Set to true to create resources, false to skip creation.')
        choice(name: 'DESTROY_RESOURCES', choices: ['no', 'yes'], description: 'Destroy previously created resources?')
        string(name: 'MIN_SIZE', defaultValue: '1', description: 'Minimum instances in ASG')
        string(name: 'MAX_SIZE', defaultValue: '5', description: 'Maximum instances in ASG')
        string(name: 'DESIRED_CAPACITY', defaultValue: '1', description: 'Desired instances in ASG')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Update Variables') {
            steps {
                script {
                    if (params.CREATE_RESOURCES) {
                        bat """
                        echo min_size = ${params.MIN_SIZE} > terraform.tfvars
                        echo max_size = ${params.MAX_SIZE} >> terraform.tfvars
                        echo desired_capacity = ${params.DESIRED_CAPACITY} >> terraform.tfvars
                        """
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                withAWS(credentials: 'aws') {
                    bat 'terraform init'
                }
            }
        }

        stage('Terraform Plan or Destroy') {
            steps {
                script {
                    if (params.CREATE_RESOURCES) {
                        echo 'Planning resource creation...'
                        withAWS(credentials: 'aws') {
                            bat 'terraform plan -var-file=terraform.tfvars'
                        }
                    } else if (params.DESTROY_RESOURCES == 'yes') {
                        echo 'Planning resource destruction...'
                        withAWS(credentials: 'aws') {
                            bat 'terraform plan -destroy -var-file=terraform.tfvars'
                        }
                    } else {
                        echo 'No operation selected for Terraform.'
                    }
                }
            }
        }

        stage('Terraform Apply or Destroy') {
            steps {
                script {
                    if (params.CREATE_RESOURCES) {
                        echo 'Applying changes to create resources...'
                        withAWS(credentials: 'aws') {
                            bat 'terraform apply -auto-approve -var-file=terraform.tfvars'
                        }
                    } else if (params.DESTROY_RESOURCES == 'yes') {
                        echo 'Applying changes to destroy resources...'
                        withAWS(credentials: 'aws') {
                            bat 'terraform destroy -auto-approve -var-file=terraform.tfvars'
                        }
                    } else {
                        echo 'No action selected for Terraform apply/destroy.'
                    }
                }
            }
        }
    }
}
