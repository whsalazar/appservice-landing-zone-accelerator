# User Guide for Reference Implementation Deployment

## Pre-requisite
1. Active Azure subscription
2. Active GitHub repository
3. (Optional) Azure DevOps or GitHub account name


## Steps
1. Clone this repository ([aka.ms/EnterpriseScale-AppService](https://aka.ms/EnterpriseScale-AppService)) to your organization/repository
   ![Clone Repo](/docs/Images/CloneRepo.png)
2. Setup authentication between Azure and GitHub. Currently there are [two options](https://docs.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows) to configure this - through OpenID Connect(coming soon) or with a service principal. Only one of the two actions is required.  
   - Use a service principal secret
        1. Open [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) in the Azure Portal or [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) locally
        2. Create a new service principal in the Azure portal for your app and assign it **Contributor** role. Replace {subscription-id}. The service principal will be created at the scope of the subscription as multiple resource groups will be created.
            ```
            az ad sp create-for-rbac --name "myApp" --role contributor \
                                --scopes /subscriptions/{subscription-id} \
                                --sdk-auth
            ```
        3. Copy the JSON object for your service principal
            ```json
            {
                "clientId": "<GUID>",
                "clientSecret": "<GUID>",
                "subscriptionId": "<GUID>",
                "tenantId": "<GUID>",
                (...)
            }
            ```
        4. Navigate to where you cloned the GitHub repository and go to **Settings** and select **Secrets**.
            ![Secrets](/docs/Images/Secrets.png)
        5. Select **New repository secrets**.
            ![New Secrets](/docs/Images/NewSecrets.png)
        6. Create a new secret called `AZURE_CREDENTIALS` with the JSON information in step 3 (in JSON format).
   - Use Azure login action with OpenID Connect (coming soon)
3. Create the following secrets with corresponding infromation:
   - `ACCOUNT_NAME` (your GitHub/Azure DevOps account name in plain text)
   - `AZURE_SUBSCRIPTION` (your Azure subscription ID in plain text)
   - `PAT` (your personal access token for Azure Devops/GitHub in plain text)
   - `VM_PW` (password for the VMs that will be created in plain text)
   - (Optional) `ACTIONS_STEP_DEBUG` (set this to true if you want [additional information](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging) running the GitHub workflows)
4. Navigate to [config.yml](../reference-implementations/LOB-ILB-ASEv3/bicep/config.yml) and modify any of the following values as required:
    
    Below are the default values in config.yml
    
    ```yaml
    AZURE_LOCATION: 'westus2'
    RESOURCE_NAME_PREFIX: 'ase-demo'
    ENVIRONMENT_TAG: 'dev'
    DEPLOYMENT_NAME: 'ase-demo-deployment'
    VM_USERNAME: 'agent'
    ACCOUNT_NAME: 'replace me in repo secrets'
    CICD_AGENT_TYPE: 'azuredevops'
    ```

   - `AZURE_LOCATION` (supported regions for the current subscription can be found with [az account list-locations](https://docs.microsoft.com/en-us/cli/azure/account?view=azure-cli-latest#az_account_list_locations))
   - `RESOURCE_NAME_PREFIX` (prefix that's added to all reosurces and resource groups that gets created. Prefix must be 12 characters or less and used to create a globaly unique name)
   - `ENVIRONMENT_TAG` (dev, uat, prod, or dr)
   - `DEPLOYMENT_NAME` (used for logging for deployment hisotry)
   - `VM_USERNAME` (user name for VMs created)
   - `ACCOUNT_NAME` (the value will be replaced with repository secret)
   - `CICD_AGENT_TYPE` (github, azuredevops, or none) 
5. Push the configuration changes to your feature branch, then create a pull request to main. This should trigger the build. Current builds can be found at **Actions** with the selected workflow (AzureBicepDeploy in this case)
   ![AzureBicepDeploy](/docs/Images/AzureBicepDeploy.png)
   Alternatively, you can also trigger a build by going to **Actions** selecting the specific workflow (ie. AzureBicepDeploy), and then selecting **Run workflow**.
   ![WorkflowDispatch](/docs/Images/WorkFlowDispatch.png)
6. A deployment for an App Service Landing Zone Accelerator reference implementation should look something similar to the following:
   ![Resource Groups](/docs/Images/ResourceGroups.png)
   Outputs from ASE Module:
   ![ASE](/docs/Images/ASE.png)
   Outputs from Shared Module:
   ![Shared](/docs/Images/Shared.png)
   Outputs from Networking Module:
   ![Networking](/docs/Images/Networking.png)
7. Sample Application Deployment to ASE using GitHub Actions (OPTIONAL)

    1. Check that self-hosted runner was successfully added to your organization/repository
        - Navigate to main page of repository
        - Under repository name, click Settings
        - In the left sidebar, click Actions, then click Runners
        - Check to ensure self-hosted runner is running. When the runner application is connected to GitHub and ready to receive jobs, you will see the following message on the github-dev VM's terminal.
            ```
            √ Connected to GitHub
            2019-10-24 05:45:56Z: Listening for Jobs
            ```

    2. Modify application deployment workflow:

        - Navigate to [deploy-sample-app.yml](../.github/workflows/deploy-sample-app.yml) and modify any of the following values as required:
    
        - Below are the default values in deploy-sample-app.yml
    
            ```yaml
            buildConfiguration: 'Release'
            resourceGroupName: 'rg-ase-ase-demo-whs-dev-westus2-001'
            dotnetVersion: '6.0.x'
            ServiceEndPointName: 'ASEServiceEndPoint'
            artifactName: 'SampleApp'
            WebAppName: 'whsasetest'
            ```
   3. Run Workflow
        - Push the configuration changes to your feature branch, then create a pull request to main. This should trigger the build. Current builds can be found at **Actions** with the deploy-sample-app workflow 
        - Alternatively, you can also trigger a build by going to **Actions** selecting the specific workflow (ie. AzureBicepDeploy), and then selecting **Run workflow**.
        
   4. Open Sample Application
        - In an App Service Environment with an internal virtual IP (VIP), the domain suffix used for app creation is .<asename>.appserviceenvironment.net. If your App Service Environment is named my-ase, and you host an app called contoso, you reach it at these URLs:
            - contoso.my-ase.appserviceenvironment.net
            - contoso.scm.my-ase.appserviceenvironment.net



