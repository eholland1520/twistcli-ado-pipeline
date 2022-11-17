## TwistCLI Integration into Azure DevOps Pipeline
The following document is provided as high level example of how to integrate Prisma Cloud TwistCLI into and Azure DevOps pipeline using either the classic graphical editor in ADO or via YAML pipeline template.

#### Pre-requisites
1. Create Access Keys
2. Create JWT

#### Create Access Keys
Settings --> Access Control --> Access keys --> Click "Add" button and select "Access Key"

![prisma-cloud-access-key-create](prisma-cloud-access-key-create.png)

![prisma-cloud-access-key-credentials.png](prisma-cloud-access-key-credentials.png)
Save the password in a secure place like Azure Keyvault for retrieval during the Azure DevOps pipeline

#### Create a JWT 
[Detailed instructions can be found here](https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA14u0000004MQyCAM&lang=en_US%E2%80%A9&refURL=http%3A%2F%2Fknowledgebase.paloaltonetworks.com%2FKCSArticleDetail)
```
curl -X POST \
            https://api.prismacloud.io/login \
            -H 'Content-Type: application/json' \
            -d '{"username":"Access-Key","password":"Secret-key"}'
```
The command above outputs a JWT that can be used for authenticating to the Prisma Cloud API when downloading the twistcli binary and accessing image scans.
```
{
           "token": "<JWT>",
           "message": "login_successful",
           "customerNames": [
           {
          "customerName": "Test",
          "tosAccepted": true
          }
         ]
        }
```

**Note**: In a production scenario step 2 can be eliminated by installing the twistcli binary on Azure self-hosted build agents ensuring that all developers have access to image scanning capabilities for their individual pipelines.

## Build Pipeline Steps
1. Retrieve access token and console url from Azure Key Vault
2. Use access key to download and install the TwistCLI binary
3. Build a docker image
4. Scan a docker image using TwistCLI (Pass/Fail)
5. If the image passes the previous step (5), push the image to ACR

#### TwistCLI Command Line
The Azure DevOps pipeline executes the following command
```
#vDownload the twistcli binary from the console
curl --progress-bar -L -k --header "authorization: Bearer __prisma-accesstoken__" __prisma-consoleurl__/api/v1/util/twistcli > twistcli; chmod a+x twistcli;

# Scan the prisma-rabbitmq docker container
./twistcli images scan --details --address __prisma-consoleurl__ --token __prisma-accesstoken__ youracr.azurecr.io/rabbitmq-prisma:prisma
```

* Graphical Editor - Use the classic graphical editor to build your pipeline in the web browser.
* Yaml - Create a yaml file that can be stored along with other source code files for the project.

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
### Image Scanner Report
**Note**: The following screenshot shows the output of an image scan from an Azure DevOps pipeline.
![TwistCLI Scanner Report](twistcli-scanner-report.png)
