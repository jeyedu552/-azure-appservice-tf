pipeline {
    agent any

    environment {
        TF_IN_AUTOMATION = "true"
    }

    stages {
        stage('Clonar Repositorio') {
            steps {
                git branch: 'main', url: 'https://github.com/jaimepsayago/azure-appservice-tf'            }
        }

        stage('Autenticación Azure') {
            steps {
                withCredentials([string(credentialsId: 'azure-service-principal', variable: 'AZURE_CREDENTIALS_JSON')]) {
                    sh '''
                        echo "$AZURE_CREDENTIALS_JSON" > azure-credentials.json

                        export ARM_CLIENT_ID=$(jq -r .clientId azure-credentials.json)
                        export ARM_CLIENT_SECRET=$(jq -r .clientSecret azure-credentials.json)
                        export ARM_TENANT_ID=$(jq -r .tenantId azure-credentials.json)
                        export ARM_SUBSCRIPTION_ID=$(jq -r .subscriptionId azure-credentials.json)

                        echo "client_id       = \\"$ARM_CLIENT_ID\\"" > terraform.tfvars
                        echo "client_secret   = \\"$ARM_CLIENT_SECRET\\"" >> terraform.tfvars
                        echo "tenant_id       = \\"$ARM_TENANT_ID\\"" >> terraform.tfvars
                        echo "subscription_id = \\"$ARM_SUBSCRIPTION_ID\\"" >> terraform.tfvars

                        az login --service-principal \
                          --username $ARM_CLIENT_ID \
                          --password $ARM_CLIENT_SECRET \
                          --tenant $ARM_TENANT_ID

                        az account set --subscription $ARM_SUBSCRIPTION_ID
                    '''
                }
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        stage('Aprobación') {
            steps {
                input message: '¿Deseas aplicar los cambios en Azure?'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }
}
