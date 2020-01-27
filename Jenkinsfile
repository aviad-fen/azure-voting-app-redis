node('master') {

    def servicePrincipalId = 'AzureKubernetesId'
    def resourceGroup = 'RTDS-QA-RG'
    def aks = 'RTDSQAAKS'
    
    
    def dockerRegistry = 'rtdsqa.azurecr.io'
    
    def imageName = "todo-app:${env.BUILD_NUMBER}"
    env.IMAGE_TAG = "${dockerRegistry}/${imageName}"
    def dockerCredentialId = 'AzureContainersRegistryId'

    def currentEnvironment = 'blue'
    def newEnvironment = { ->
        currentEnvironment == 'blue' ? 'green' : 'blue'
    }

    stage('SCM') {
        checkout scm
    }
    
    stage('Build') {
        withCredentials([azureServicePrincipal(servicePrincipalId)]) {
            sh """
                az login --service-principal -u "\$AZURE_CLIENT_ID" -p "\$AZURE_CLIENT_SECRET" -t "\$AZURE_TENANT_ID"
                az account set --subscription "\$AZURE_SUBSCRIPTION_ID"
                az logout
            """
        }
    }
    
    stage('Docker Image') {
        withDockerRegistry([credentialsId: dockerCredentialId, url: "https://${dockerRegistry}"]) {
            dir('target') {
                sh """
                    cp -f azure-vote/Dockerfile .
                    docker build -t "${env.IMAGE_TAG}" .
                    docker push "${env.IMAGE_TAG}"
                """
            }
        }
    }
    
    stage('Deploy') {
        // check the current active environment to determine the inactive one that will be deployed to

        withCredentials([azureServicePrincipal(servicePrincipalId)]) {
            // fetch the current service configuration
            sh """
              
              az login --service-principal -u "\$AZURE_CLIENT_ID" -p "\$AZURE_CLIENT_SECRET" -t "\$AZURE_TENANT_ID"
              az account set --subscription "\$AZURE_SUBSCRIPTION_ID"
              az aks get-credentials --resource-group "${resourceGroup}" --name "${aks}" --admin --file kubeconfig
              az logout
              kubectl --kubeconfig kubeconfig apply -f azure-vote-all-in-one-redis.yaml
              
            """
        }
    }

    stage('Post-clean') {
        sh '''
          rm -f kubeconfig
        '''
    }
    
    
}
