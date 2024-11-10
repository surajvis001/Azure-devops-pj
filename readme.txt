trigger: none

pool: selfhosted-pool-01

steps:
- checkout: self
- script: |
  displayName: fetching current directory
    echo "Current Directory:"
    pwd
    echo "Listing files:"
    ls -la  
- task: AzureCLI@2
  inputs:
    azureSubscription: 'acr-aks-spn'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      tag = $(Build.BuildId)
      docker build -t dockercontainertest.azurecr.io/web-game:latest
      az acr login -n dockercontainertest
      docker push dockercontainertest.azurecr.io/web-game:latest

- script: cat 01_kubernetes_aks/app-deploy.yaml

- task: Kubernetes@1
  inputs:
    connectionType: 'Azure Resource Manager'
    azureSubscriptionEndpoint: 'acr-aks-spn'
    azureResourceGroup: 'testing-resources'
    kubernetesCluster: 'aks-test-practice'
    namespace: 'default'
    command: 'apply'
    useConfigurationFile: true
    configuration: '01_kubernetes_aks'
    secretType: 'dockerRegistry'
    containerRegistryType: 'Azure Container Registry'
    azureSubscriptionEndpointForSecrets: 'acr-aks-spn'
    azureContainerRegistry: 'dockercontainertest.azurecr.io/web-game'
    forceUpdate: false
