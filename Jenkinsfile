pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'reactAppAryan01'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aryanjindal724/MyWebAPIWithJenkinsInReactJS.git'
            }
        }

        stage('Terraform Init & Apply') {
            steps {
                dir('Terraform_module') {
                    withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                        bat '''
                        set ARM_CLIENT_ID=%AZURE_CLIENT_ID%
                        set ARM_CLIENT_SECRET=%AZURE_CLIENT_SECRET%
                        set ARM_SUBSCRIPTION_ID=%AZURE_SUBSCRIPTION_ID%
                        set ARM_TENANT_ID=%AZURE_TENANT_ID%
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        terraform init
                        terraform plan
                        terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Build React App') {
            steps {
                dir('reactapp-for-jenkinsandterraform') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Deploy to Azure') {
            steps {
                dir('reactapp-for-jenkinsandterraform') {
                    withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                        bat '''
                        powershell Compress-Archive -Path build\\* -DestinationPath build.zip -Force
                        az webapp deployment source config-zip --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src build.zip
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'React App deployed to Azure!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}
