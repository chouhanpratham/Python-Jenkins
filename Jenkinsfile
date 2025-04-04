pipeline {
    agent any
    
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-python'
        APP_SERVICE_NAME = 'python-webapp-service-pratham1'
        PYTHON_VERSION = '3.11'
        PYTHON_PATH = '/opt/anaconda3/bin/python3'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/chouhanpratham/Python-Jenkins.git'
            }
        }
        
        stage('Build') {
            steps {
                sh '$PYTHON_PATH --version'
                sh '$PYTHON_PATH -m pip install --upgrade pip'
                sh '$PYTHON_PATH -m pip install -r requirements.txt'
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    
                    // Login to Azure
                    sh 'az login --service-principal -u "$AZURE_CLIENT_ID" -p "$AZURE_CLIENT_SECRET" --tenant "$AZURE_TENANT_ID"'
                    
                    // Create resource group
                    sh 'az group create --name $RESOURCE_GROUP --location eastus'
                    
                    // Create app service plan
                    sh 'az appservice plan create --name ${APP_SERVICE_NAME}-plan --resource-group $RESOURCE_GROUP --sku B1 --is-linux'
                    
                    // Create web app
                    sh 'az webapp create --resource-group $RESOURCE_GROUP --plan ${APP_SERVICE_NAME}-plan --name $APP_SERVICE_NAME --runtime "PYTHON|$PYTHON_VERSION"'
         
                    // Configure web app
                    sh 'az webapp config set --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --startup-file "gunicorn --bind=0.0.0.0 --timeout 600 app:app"'

                    // Create deployment package
                    sh 'zip -r deploy.zip ./*'

                    // Deploy application
                    sh 'az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path ./deploy.zip --timeout 1800'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
