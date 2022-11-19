# Azure DevOps Pipeline with TwistCLI Image Scanning
The following document is provided as high level example of how to integrate Prisma Cloud TwistCLI into and Azure DevOps pipeline using either the classic graphical editor in ADO or via YAML pipeline template.

#### Steps 
1. Create a Prisma Cloud Service Account with a DevOps role in Prisma Cloud Compute.
2. Install the Prisma Compute plugin in your Azure DevOps Organization.
3. Build an Azure DevOps pipeline to scan a docker image for vulnerabilities with TwistCli.
5. View the vulnerability report produced by the TwistCli image scan.

## 1. Create Prisma Cloud Service Account
Settings --> Access Control --> User --> Click "Add" button and select "Service Account". Provide a name for the service account and select "DevOps-role" to enable scanning permissions for the service account.
<img src="images/prisma-cloud-service-account-devops-role.png" width="45%">

After creating the service account you will be prompted to create an access key for the service account. Provide a name for the service account and click "Next".
![prisma-cloud-access-key-details](images/prisma-cloud-access-key-details.png)

After the access key has been created you are give the access key ID and the access key secret token. These values are the user and password that you will need when authenticating to the consolfe from your Azure DevOps pipeline. Be sure to save these values in a secure place (Azure Keyvault).
![prisma-cloud-access-key-results](images/prisma-cloud-access-key-results.png)

Now that you have a service account user and a user/password for authentication you can move on to Azure DevOps.

## 2. Install the Prisma Compute plugin
The Prisma Compute plugin is available from the Azure Marketplace:
https://marketplace.visualstudio.com/items?itemName=PrismaCloud.build-release-task

## 3. Build an Azure Devops Pipeline with TwistCli image scanning
1. Retrieve access token and console url from Azure Key Vault
2. Use access key to download and install the TwistCLI binary
3. Build a docker image
4. Scan a docker image using TwistCLI (Pass/Fail)
5. If the image passes the previous step (5), push the image to ACR

**Note**: In a production scenario step 2 can be eliminated by installing the twistcli binary on Azure self-hosted build agents ensuring that all developers have access to image scanning capabilities for their individual pipelines.

#### TwistCLI Command Line
The Azure DevOps pipeline executes the following twistcli commands during build pipeline execution.

Reference: https://docs.paloaltonetworks.com/prisma/prisma-cloud/prisma-cloud-admin-compute/tools/twistcli_scan_images

```
#vDownload the twistcli binary from the console (Task: Install TwistCLI)
curl --progress-bar -L -k --header "authorization: Bearer __prisma-accesstoken__" __prisma-consoleurl__/api/v1/util/twistcli > twistcli; chmod a+x twistcli;

# Scan the prisma-rabbitmq docker container (Task: Scan RabbitMQ Image)
./twistcli images scan --details --address __prisma-consoleurl__ --token __prisma-accesstoken__ youracr.azurecr.io/rabbitmq-prisma:prisma
```

The build pipeline can either be executed from the classic web based editor or via yaml template.
* Graphical Editor - Use the classic graphical editor to build your pipeline in the web browser.
* Yaml - Create a yaml file that can be stored along with other source code files for the project.

#### Classic Editor Pipeline
![azure-devops-pipeline-editor](azure-devops-pipeline-editor.png)

#### YAML Pipeline
```
# Variable Group 'prismacloud' was defined in the Variables tab
resources:
  repositories:
  - repository: self
    type: git
    ref: refs/heads/prisma
jobs:
- job: Job_1
  displayName: Agent job 1
  pool:
    vmImage: ubuntu-18.04
  steps:
  - checkout: self
    fetchDepth: 1
  - task: replacetokens@5
    displayName: Get Prisma Cloud Secrets From Azure KeyVault
    inputs:
      rootDirectory: scripts
      targetFiles: '**/*.sh'
      tokenPattern: rm
      keepToken: true
  - task: AzureCLI@2
    displayName: Install TwistCLI
    inputs:
      connectedServiceNameARM: yyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
      scriptType: bash
      scriptPath: scripts/install-twistcli.sh
      inlineScript: 'curl --progress-bar -L -k --header "authorization: Bearer [__prisma-accesstoken__]" [__prisma-consoleurl__]/api/v1/util/twistcli > twistcli; chmod a+x twistcli;'
  - task: Docker@2
    displayName: Create RabbitMQ Image
    inputs:
      containerRegistry: xxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxx
      repository: rabbitmq-prisma
      command: build
      tags: prisma
  - task: AzureCLI@2
    displayName: Scan RabbitMQ Image
    inputs:
      connectedServiceNameARM: yyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
      scriptType: bash
      scriptPath: scripts/scan-image.sh
      inlineScript: ./twistcli images scan --details --address https://us-east1.cloud.twistlock.com/us-1-111573457 --token [__prisma-accesstoken__] youracr.azurecr.io/rabbitmq-prisma:prisma
      scriptArguments: __prisma-consoleurl__
      failOnStandardError: true
  - task: Docker@2
    displayName: Push RabbitMQ Image to ACR
    inputs:
      containerRegistry: xxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxx
      repository: rabbitmq-prisma
      command: push
      tags: prisma
```
4. ## TwistCLI Image Scanner Report
**Note**: The following screenshot shows the output of an image scan from an Azure DevOps pipeline.
![TwistCLI Scanner Report](twistcli-scanner-report.png)
