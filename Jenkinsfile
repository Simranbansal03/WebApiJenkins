// For terraform
pipeline {
    agent any

    environment {
        ARM_CLIENT_ID       = credentials('AZURE_CLIENT_ID')
        ARM_CLIENT_SECRET   = credentials('AZURE_CLIENT_SECRET')
        ARM_SUBSCRIPTION_ID = credentials('AZURE_SUBSCRIPTION_ID')
        ARM_TENANT_ID       = credentials('AZURE_TENANT_ID')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Simranbansal03/WebApiJenkins', branch: 'master'
            }
        }

        stage('Terraform Init') {
            steps {
                bat 'terraform init'
            }
        }

        stage('Terraform Plan') {
            steps {
                bat '''
                    terraform plan ^
                      -var client_id=%ARM_CLIENT_ID% ^
                      -var client_secret=%ARM_CLIENT_SECRET% ^
                      -var tenant_id=%ARM_TENANT_ID% ^
                      -var subscription_id=%ARM_SUBSCRIPTION_ID%
                    '''
            }
        }

        stage('Terraform Apply') {
            steps {
                bat '''
                terraform apply -auto-approve ^
                  -var client_id=%ARM_CLIENT_ID% ^
                  -var client_secret=%ARM_CLIENT_SECRET% ^
                  -var tenant_id=%ARM_TENANT_ID% ^
                  -var subscription_id=%ARM_SUBSCRIPTION_ID%
                '''
            }
        }
         stage('Build .NET App') {
            steps {
                dir('WebApiJenkins') { // Adjust to your .NET project folder
                    bat 'dotnet publish -c Release -o publish'
                }
            }
        }

       stage('Deploy') {
    steps {
        withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
            bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
            bat "powershell Compress-Archive -Path ./publish/* -DestinationPath ./publish.zip -Force"
            bat "az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path ./publish.zip --type zip"
        }
    }
}
    
}

