# AzureDevOps-GCP

Make the necessary configuration in azure devops to perform automation in the Google Cloud.

### DOCUMENTATION:

https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch
https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/download-secure-file?view=azure-devops

In this video a good practice to be able to connect to our respective cloud and project in the Google cloud. I will also tell you about a practice that I found configured in the past and I do not recommend for security issues, we will evaluate its pros and cons

![image](https://user-images.githubusercontent.com/59833468/199477246-de85905d-0b20-4338-a495-677fdb64245e.png)

Required Components
* GCP account (free tier) or with enough balance to do these tests
* 2 service accounts
  * service-account-total-azure Owner Full access to all resources.
  * service-account-pub-sub-azure Viewer access to pub sub.
* Account in Azure devops and concepts of pipelines
* Extension "Google Cloud SDK Extension for Azure DevOps"
* Declarative Pipeline Code

# How do we create a service account?

https://cloud.google.com/iam/docs/creating-managing-service-accounts?hl=en

What are service accounts? A service account is a special type of account that is used by an application or virtual machine (VM) instance, not by a person. Applications use service accounts to make authorized API calls.

We enter the GCP web panel
Then we access through the top menu to IAM & admin
Service Accounts
We press the upper button of "+ CREATE SERVICE ACCOUNT" We assign permissions similar to these

# Extensión "Google Cloud SDK Extension for Azure DevOps"

Azure devops extension: It allows us to extend the functionalities of the CI / CD tool and in this case it will allow us to have the google SDK client in the agents where the automations will be executed.

https://marketplace.visualstudio.com/items?itemName=laurensknoll.google-cloud-sdk-tasks

![image](https://user-images.githubusercontent.com/59833468/199479515-045468bb-e969-401c-bce2-18b278ad634f.png)


# Commands that we will use once connected to the cloud

  gcloud projects list
  gcloud compute instance-groups list-instances
  gcloud app instances list
  gcloud bigtable instances list
  gcloud compute instances list
  gcloud filestore instances list
  gcloud redis instances list
  gcloud spanner instances list
  gcloud sql instances list
  gcloud compute instance-groups list

configuration of service account in Azure Devops
Following the following steps we will securely configure the Google Cloud credentials called "service account" with which we will connect to the project in the cloud.

We get our service account file "service-account-total-control-azure.yml"
Inside Azure Devops we go to Pipelines/Library/Secure files/
We upload the file "service-account-total-control-azure.yml, we allow it to be used by all pipelines.

Pipeline that we will use
Service account service-account-total-control

trigger: none

pool: 
  vmImage: 'ubuntu-latest'

variables:
        GOOGLE_PROJECT_ID: "ivory-honor-272915" 
        GOOGLE_PROJECT_NAME: "Proyecto-ideas-extraordinarias"
        GOOGLE_APPLICATION_CREDENTIALS: '$(service_account_total_azure.secureFilePath)'
        GOOGLE_CLOUD_KEYFILE_JSON: '$(service_account_total_azure.secureFilePath)'

stages:
  - stage: "GCP_Connection"
    displayName: "Connection to Google Cloud Project"
    jobs:
      - job: "Configure_Service_Account"
        displayName: "Configure Service Account"
        steps:
          - task: DownloadSecureFile@1
            name: 'service_account_total_azure'
            displayName: 'Download service account'
            inputs:
              secureFile: 'service-account-total-azure.json'
          - bash: 'echo $GOOGLE_PROJECT_ID'
          - bash: 'gcloud --version'
            name:  "version"
          - bash: 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
            name: "Active_Service"
          - bash: 'gcloud config set project $GOOGLE_PROJECT_ID'
          - bash: 'gcloud config list'
          - bash: 'gcloud projects list'
          - bash: 'gcloud compute networks list'

Service account pub sub
trigger: none

pool: 
  vmImage: 'ubuntu-latest'

variables:
        GOOGLE_PROJECT_ID: "ivory-honor-272915" 
        GOOGLE_PROJECT_NAME: "Proyecto-ideas-extraordinarias"
        GOOGLE_APPLICATION_CREDENTIALS: '$(service_account_pubsub_azure.secureFilePath)'
        GOOGLE_CLOUD_KEYFILE_JSON: '$(service_account_pubsub_azure.secureFilePath)'

stages:
  - stage: "GCP_Connection"
    displayName: "Connection to Google Cloud Project"
    jobs:
      - job: "Configure_Service_Account"
        displayName: "Configure Service Account"
        steps:
          - task: DownloadSecureFile@1
            name: 'service_account_pubsub_azure'
            displayName: 'Download service account'
            inputs:
              secureFile: 'service-account-pub-sub-azure.json'
          - bash: 'echo $GOOGLE_PROJECT_ID'
          - bash: 'gcloud --version'
            name:  "version"
          - bash: 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
            name: "Active_Service"
          - bash: 'gcloud config set project $GOOGLE_PROJECT_ID'
          - bash: 'gcloud pubsub topics list'
          - bash: 'gcloud projects list'
          - bash: 'gcloud compute networks list'
