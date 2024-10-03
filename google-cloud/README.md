# Google Cloud Platform

**Contents**:

- [Google Cloud Platform](#google-cloud-platform)
- [Install `gcloud`](#install-gcloud)
  - [Debian/Ubuntu](#debianubuntu)
  - [Create `gcloud` Configurations](#create-gcloud-configurations)
- [Authenticate](#authenticate)
  - [Using Client Libraries](#using-client-libraries)
  - [Using `gcloud` CLI](#using-gcloud-cli)
  - [Using Service Account Impersonation](#using-service-account-impersonation)
- [Set Up Application Default Credentials (ADC) for `gcloud`](#set-up-application-default-credentials-adc-for-gcloud)
  - [For Local Development Environment](#for-local-development-environment)
    - [Use your Google Account](#use-your-google-account)
    - [Impersonate a Service Account](#impersonate-a-service-account)
  - [For Google Cloud Services that support attaching a Service Account](#for-google-cloud-services-that-support-attaching-a-service-account)
- [Control Access to Resources](#control-access-to-resources)
  - [Manage Access to Projects, Folders and Organisations](#manage-access-to-projects-folders-and-organisations)
    - [Manage Access](#manage-access)
    - [View Current Access](#view-current-access)
    - [Grant a Single Role](#grant-a-single-role)
    - [Revoke a Single Role](#revoke-a-single-role)
    - [Grant/Revoke Multiple Roles Programmatically](#grantrevoke-multiple-roles-programmatically)
  - [Manage Access to Service Accounts](#manage-access-to-service-accounts)
    - [View Current Access](#view-current-access-1)
    - [Grant a Single Role](#grant-a-single-role-1)
    - [Revoke a Single Role](#revoke-a-single-role-1)
    - [Grant/Revoke Multiple Roles Programmatically](#grantrevoke-multiple-roles-programmatically-1)
- [Manage `gcloud` CLI Configurations](#manage-gcloud-cli-configurations)
  - [Set a Default Configuration](#set-a-default-configuration)
  - [Create a Configuration](#create-a-configuration)
  - [List Configurations](#list-configurations)
  - [Set Configuration Properties](#set-configuration-properties)
  - [View Configuration Properties](#view-configuration-properties)
  - [Delete a Configuration](#delete-a-configuration)

---
# Install `gcloud`

Source: https://cloud.google.com/sdk/docs/install#deb

## Debian/Ubuntu

* Update packages and install `apt-transport-https` and `curl`:
  
  ```
  $ sudo apt-get update
  $ sudo apt-get install apt-transport-https ca-certificates gnupg curl
  ```

* Import the Google Cloud public key:
  
  ```
  $ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
  ```

* Add the `gcloud` CLI distribution URI as a package source:

  ```
  echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
  ```

* Update and install the `gcloud` CLI:
  
  ```
  sudo apt-get update && sudo apt-get install google-cloud-cli
  ```

* Install any **additional components**:

  For example:

  ```
  sudo apt-get install google-cloud-cli-app-engine-python
  ```

---
## Create `gcloud` Configurations

* Create a new configuration:
  
  ```
  gcloud config configurations create [configuration_name]
  ```

  For example:

  ```
  gcloud config configurations create sebastiancg
  ```

  - You should login to your respective Google Cloud account
  afterwards:

    ```
    gcloud auth login
    ```

* Set a project for your configuration:
  
  ```
  gcloud config set project [project_id]
  ```

* Get the ID of a project:
  
  ```
  gcloud config get-value project
  ```

* List all the configurations:
  
  ```
  gcloud config configurations list
  ```

* Activate a configuration:
  
  ```
  gcloud config configurations activate [configuration_name]
  ```

* Delete a configuration:

  ```
  gcloud config configurations delete [configuration_name]
  ```

* Describe a configuration:
  
  ```
  gcloud config configurations describe [configuration_name]
  ```

---
# Authenticate

Use the following diagram to help you choose an authentication 
method:

  ![](https://cloud.google.com/static/docs/authentication/images/authn-tree.svg)

## Using Client Libraries

Client libraries allows you to access Google Cloud APIs using 
a supported language: e.g., Python, Java or Node.js

* Set up 
[Application Default Credentials (ADC)](#set-up-application-default-credentials-adc-for-gcloud)
for the environment where your application is running.

  - The client library automatically checks for and uses the 
  credentials provided to ADC to authenticate to the APIs.

  - **Local environment**: Use your user credentials or a service 
  account impersonation

  - **Production environment**: Attach a service account
  
  - The User or Service account to use must have the required 
  permissions for the project

* Install the client libraries:
  
  - For Python:
  
  ```
  pip install --upgrade google-cloud-[LIBRARY_NAME]
  ```

  For example:

  ```
  pip install --upgrade google-cloud-storage
  ```

  - Check **all Python Cloud Client Libraries** at 
  https://cloud.google.com/python/docs/reference

* Create a code that interacts with Google Cloud Client libraries:
  
  An example for Cloud Storage:

  ```
  from google.cloud import storage

  def authenticate_implicit_with_adc(project_id="your-google-cloud-project-id"):
      """
      When interacting with Google Cloud Client libraries, the library can auto-detect the
      credentials to use.

      // TODO(Developer):
      //  1. Before running this sample,
      //  set up ADC as described in https://cloud.google.com/docs/authentication/external/set-up-adc
      //  2. Replace the project variable.
      //  3. Make sure that the user account or service account that you are using
      //  has the required permissions. For this sample, you must have "storage.buckets.list".
      Args:
          project_id: The project id of your Google Cloud project.
      """

      # This snippet demonstrates how to list buckets.
      # *NOTE*: Replace the client created below with the client required for your application.
      # Note that the credentials are not specified when constructing the client.
      # Hence, the client library will look for credentials using ADC.
      storage_client = storage.Client(project=project_id)
      buckets = storage_client.list_buckets()
      print("Buckets:")
      for bucket in buckets:
          print(bucket.name)
      print("Listed all storage buckets.")
  ```

## Using `gcloud` CLI

* **Using User Credentials**:
  
  ```
  $ gcloud init
  $ gcloud auth login
  ```

* **Using Workforce Identity Login**:
  
  ```
  $ gcloud config set auth/login_config_file [WORKFORCE_ID_LOGIN_CONF_FILE]
  $ gcloud auth login
  ```

* **Using Workload Identity Login for Service Accounts**:
  
  ```
  $ gcloud auth login --cred-file=[WORKLOAD_ID_CREDENTIAL_FILE]
  ```

## Using Service Account Impersonation

In Service Account Impersonation, you start with an authenticated 
principal (your user account or a service account) and request 
short-lived credentials for a service account

* The authenticated principal **must have the necessary permissions** 
to impersonate the service account

* Service account impersonation is **more secure than using a 
service account key** because:

  - Service account impersonation requires a prior authenticated 
  identity
  
  - Credentials that are created by using impersonation do not 
  persist

* Enable **Service Account Credentials API** for the project

* Use impersonation with the `gcloud` CLI by default:
  
  ```
  gcloud config set auth/impersonate_service_account [SERVICE_ACCT_EMAIL]
  ```

  - `gcloud` CLI requests short-lived credentials for the specified 
  service account and uses them to authenticate to the API and 
  authorize the access to the resource for every command

* Use impersonation for a specific `gcloud` CLI command:
  
  An example for Cloud Storage:

  ```
  gcloud storage buckets list --impersonate-service-account=[SERVICE_ACCT_EMAIL]
  ```

# Set Up Application Default Credentials (ADC) for `gcloud`

Application Default Credentials (ADC) is a strategy used by the 
authentication libraries to automatically find credentials based 
on the application environment.

You set up ADC by providing credentials to ADC in the environment 
where your code is running.

* Run `gcloud init` to initialise Google Cloud CLI:
  
  ```
  gcloud init
  ```

## For Local Development Environment

Use **your user account** when running your code in a local
development environment:

**Important**:

* ADC configuration with a user account might not work for some 
methods and APIs: e.g., Cloud Translation or Cloud Vision

* The local ADC file contains your refresh token. Any user with 
access to your file system can use it!

  - To **revoke local credentials**:
  
    ```
    gcloud auth application-default revoke
    ```

* Your local ADC file is associated with your user account, not 
your `gcloud` CLI configuration.

### Use your Google Account

* In a local shell, create local authentication credentials for 
your user account:

  ```
  gcloud auth application-default login
  ```

  - A sign-in screen appears. After you sign in, your credentials 
  are stored in the local credential file used by ADC.

### Impersonate a Service Account

You can configure ADC with credentials from a service account by 
using **service account impersonation** or by using a service 
account key.

* You must have the **Service Account Token Creator** 
(`roles/iam.serviceAccountTokenCreator`) IAM role on the service 
account you are impersonating.

* Create a service account:
  
  ```
  gcloud iam service-accounts create [SERVICE_ACCOUNT_NAME]
  ```

* Provide access to your project and resources, and grant a role:
  
  ```
  gcloud projects add-iam-policy-binding [PROJECT_ID] --member="serviceAccount:[SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com" --role=[ROLE]
  ```

* Grant the required role to the principal that will attach the 
service account to other resources:
  
  ```
  gcloud iam service-accounts add-iam-policy-binding [SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com --member="serviceAccount:[SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com" --role=roles/iam.serviceAccountUser
  ```

* Check the role bindings for the service account:

  ```
  gcloud iam service-accounts get-iam-policy [SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com
  ```

* Use service account impersonation to create a local ADC file:
  
  ```
  gcloud auth application-default login --impersonate-service-account [SERVICE_ACCT_EMAIL]
  ```

  - Credentials are automatically found by the authentication 
  libraries. 

* Stop using impersonated service account credentials with the 
`gcloud` CLI by default:

  ```
  gcloud config unset auth/impersonate_service_account
  ```
  ```
  gcloud auth application-default login
  ```

## For Google Cloud Services that support attaching a Service Account

Some Google Cloud services (e.g., Compute Engine, App Engine, etc.)
support **attaching a user-managed service account**.

Attaching a user-managed service account is the **preferred way 
to provide credentials to ADC for production code** running on 
Google Cloud.

* Using the **default service account is NOT recommended**, 
because by default the default service account is highly privileged, 
which **violates the principle of least privilege**.

* You need to determine the roles required for your service 
accounts:
  
  https://cloud.google.com/iam/docs/choose-predefined-roles#identify-permissions

* Create a service account:
  
  ```
  gcloud iam service-accounts create [SERVICE_ACCOUNT_NAME]
  ```

* Provide access to your project and your resources, and grant 
a role to the service account:

  - You can use the same command to grant any other role
  
  ```
  gcloud projects add-iam-policy-binding [PROJECT_ID] --member="serviceAccount:[SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com" --role=[ROLE]
  ```

* Grant the required role to the principal that will attach the 
service account to other resources:

  ```
  gcloud iam service-accounts add-iam-policy-binding [SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com --member="serviceAccount:[SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com" --role=roles/iam.serviceAccountUser
  ```

* Check the granted roles for the service account:
  
  ```
  gcloud projects get-iam-policy [PROJECT_ID] --flatten="bindings[].members" --format="table(bindings.role)" --filter="bindings.members:[[SERVICE_ACCT_NAME]@[PROJECT_ID].iam.gserviceaccount.com]"
  ```

---
# Control Access to Resources

## Manage Access to Projects, Folders and Organisations

* In Identity and Access Management (IAM), access is granted 
through allow policies, also known as IAM policies. 

* An allow policy is attached to a Google Cloud resource. 

* Each allow policy contains a collection of Role Bindings that 
associate one or more principals, such as users or service accounts, 
with an IAM role.

* These Role Bindings grant specified roles on the resource and its
descendants.

Before you begin:

* Enable the Resource Manager API

* Set up Authentication

### Manage Access

* **Projects**:

  - Role: **Project IAM Admin** (`roles/resourcemanager.projectIamAdmin`)

* **Folders**:

  - Role: **Folder Admin** (`roles/resourcemanager.folderAdmin`)

* **Organisation**:

  - Role: **Organization Admin** (`roles/resourcemanager.organizationAdmin`)

### View Current Access

* Get the allow policy for the resource:
  
  ```
  gcloud [RESOURCE_TYPE] get-iam-policy [RESOURCE_ID] --format=[FORMAT] > [PATH]
  ```
  - `RESOURCE_TYPE`: Use `projects`, `resource-manager folders` or 
  `organizations`

  - `FORMAT` can be `json` or `yaml`
  
  An example for a project:

  ```
  gcloud projects get-iam-policy my-project --format=json > ~/policy.json
  ```

### Grant a Single Role

* In Linux:
  
  ```
  gcloud [RESOURCE_TYPE] add-iam-policy-binding [RESOURCE_ID] \
    --member=[PRINCIPAL] --role=[ROLE_NAME] \
    --condition=[CONDITION]
  ```

* In Windows (Powershell):
  
  ```
  gcloud [RESOURCE_TYPE] add-iam-policy-binding [RESOURCE_ID] `
    --member=[PRINCIPAL] --role=[ROLE_NAME] `
    --condition=[CONDITION]
  ```
  
  - `RESOURCE_TYPE`: Use `projects`, `resource-manager folders` or 
  `organizations`
  
  - `PRINCIPAL`: Usually has the form: `[PRINCIPAL_TYPE]:[ID]`, 
  e.g., `user:my-user@example.com`

  - `ROLE_NAME`: Use any of the formats:
    
    * Predefined roles: `roles/[SERVICE].[IDENTIFIER]`
    
    * Project-level custom roles: 
    `projects/[PROJECT_ID]/roles/[IDENTIFIER]`

    * Organisation-level custom roles: 
    `organizations/[ORG_ID]/roles/[IDENTIFIER]`

  - `CONDITION`: The condition to add to the role binding. If you 
  don't want to add a condition, use the value `None`

### Revoke a Single Role

  ```
  gcloud RESOURCE_TYPE remove-iam-policy-binding [RESOURCE_ID] --member=[PRINCIPAL] --role=[ROLE_NAME]
  ```

### Grant/Revoke Multiple Roles Programmatically

Make large-scale access changes for multiple principals, use the
*read-modify-write* pattern:

1. Read the current allow policy by calling `get-iam-policy` as
in the [View Current Access](#view-current-access) section

2. Edit the allow policy using a text editor:
  
    ```
    {
      "bindings": [
        {
          "members": [
            "user:sebastian.caceres.g@gmail.com"
          ],
          "role": "roles/owner"
        }
      ],
      "etag": "BwYWsi4AEKc=",
      "version": 1
    }
    ```

  - Grant roles by adding principals to "members"
  
  - Add roles to the allow policy by adding new "bindings"
  
  - Revoke roles by deleting "members" or "bindings"

3. Write the updated allow policy by calling:
   
    ```
    gcloud [RESOURCE_TYPE] set-iam-policy [RESOURCE_ID] [PATH]
    ```

    An example for a project:

    ```
    gcloud projects set-iam-policy my-project ~/policy.json
    ```

## Manage Access to Service Accounts

### View Current Access

* Get the allow policy for the service account:
  
  ```
  gcloud iam service-accounts get-iam-policy [SERVICE_ACCT_ID] --format=[FORMAT] > [PATH]
  ```

  - `SERVICE_ACCT_ID`: he service account's email address in the 
  form `[SERVICE_ACCT_ID]@[PROJECT_ID].iam.gserviceaccount.com`, 
  or the service account's unique numeric ID

  - `FORMAT`: `json` or `yaml`
  
  For example:

  ```
  gcloud iam service-accounts get-iam-policy my-service-account --format json > ~/policy.json
  ```

### Grant a Single Role

* Grant a single role to a principal:
  
  ```
  gcloud iam service-accounts add-iam-policy-binding [SERVICE_ACCT_ID] --member=[PRINCIPAL] --role=[ROLE_NAME] --condition=[CONDITION]
  ```

  For example:

  ```
  gcloud iam service-accounts add-iam-policy-binding my-service-account@my-project.iam.gserviceaccount.com \
    --member=user:my-user@example.com --role=roles/iam.serviceAccountUser
  ```

### Revoke a Single Role

* Revoke a single role from a principal:
  
  ```
  gcloud iam service-accounts remove-iam-policy-binding [SERVICE_ACCT_ID] --member=[PRINCIPAL] --role=[ROLE_NAME]
  ```

  For example:

  ```
  gcloud iam service-accounts remove-iam-policy-binding my-service-account@my-project.iam.gserviceaccount.com --member=user:my-user@example.com --role=roles/iam.serviceAccountUser
  ```

### Grant/Revoke Multiple Roles Programmatically

Similar to the previous section, it's recommended to make 
large-scale access changes for multiple principals, use the 
*read-modify-write* pattern:

1. Get the current allow policy:

  ```
  gcloud iam service-accounts get-iam-policy [SERVICE_ACCT_ID] --format=[FORMAT] > [PATH]
  ```

2. Edit the allow policy using a text editor:
  
    ```
    {
      "bindings": [
        {
          "members": [
            "user:sebastian.caceres.g@gmail.com"
          ],
          "role": "roles/owner"
        }
      ],
      "etag": "BwYWsi4AEKc=",
      "version": 1
    }
    ```

  - Grant roles by adding principals to "members"
  
  - Add roles to the allow policy by adding new "bindings"
  
  - Revoke roles by deleting "members" or "bindings"

3. Write the updated allow policy by calling:
   
    ```
    gcloud iam service-accounts set-iam-policy [SERVICE_ACCT_ID] [PATH]
    ```

    An example for a project:

    ```
    gcloud iam service-accounts set-iam-policy my-service-account@my-project.iam.gserviceaccount.com \
    ~/policy.json
    ```

---
# Manage `gcloud` CLI Configurations

Properties that are commonly stored in configurations include:

* Default Compute Engine zone

* Verbosity level

* Usage reporting

* Project ID

* Active user or service account

Configurations are stored in your user config directory (typically 
`~/.config/gcloud` on MacOS and Linux, or `%APPDATA%\gcloud` on 
Windows).

You can have multiple configurations and choose to switch between 
them or run commands using a specific configuration (using the 
`--configuration` flag)

## Set a Default Configuration

`gcloud` CLI starts you off with a single configuration named 
`default`

* Set properties in your configuration:
  
  ```
  gcloud config set [PROPERTY] [VALUE]
  ```
  
  For example:

  ```
  gcloud config set project [PROJECT_ID]
  ```
  ```
  gcloud config set compute/zone [ZONE_NAME]
  ```

## Create a Configuration

* Create a configuration:
  
  ```
  gcloud config configurations create [CONFIG_NAME]
  ```

* Activate the new configuration before using it:
  
  ```
  gcloud config configurations activate [CONFIG_NAME]
  ```

* Check the properties of the **Active** configuration:

  ```
  gcloud config list 
  ```

## List Configurations

* List the configurations in your `gcloud` CLI installation:
  
  ```
  gcloud config configurations list
  ```
  ```
  NAME         IS_ACTIVE     ACCOUNT            PROJECT               DEFAULT_ZONE  DEFAULT_REGION
  default      False         user@gmail.com     example-project-1     us-east1-b    us-east1
  project-1    False         user@gmail.com     example-project-2     us-east1-c    us-east1
  project-2    True          user@gmail.com     example-project-3     us-east1-b    us-east1
  ```

## Set Configuration Properties

* Set and unset the properties in the active configuration:
  
  ```
  $ gcloud config set project [PROJECT]
  $ gcloud config unset project
  ```

* Properties can also be set via environment variables:
  
  - In Linux:
    
    ```
    $ export CLOUDSDK_CORE_PROJECT=[PROJECT_NAME]
    $ export CLOUDSDK_COMPUTE_REGION=[REGION]
    $ export CLOUDSDK_COMPUTE_ZONE=[ZONE]
    ```

  - In Windows:

    ```
    > set CLOUDSDK_CORE_PROJECT=[PROJECT_NAME]
    > set CLOUDSDK_COMPUTE_REGION=[REGION]
    > set CLOUDSDK_COMPUTE_ZONE=[ZONE]
    ```

* Unset using:
  
  - In Linux:
    
    ```
    $ unset CLOUDSDK_COMPUTE_REGION
    $ unset CLOUDSDK_COMPUTE_ZONE
    ```
  
  - In Windows:
    
    ```
    > set CLOUDSDK_COMPUTE_REGION=
    > set CLOUDSDK_COMPUTE_ZONE=
    ```

## View Configuration Properties

* View the properties in a configuration:
  
  ```
  gcloud config configurations describe [CONFIG_NAME]
  ```

## Delete a Configuration

* Delete a configuration:
  
  ```
  gcloud config configurations delete [CONFIG_NAME]
  ```

* You can't delete the active configuration. Use gcloud config 
configurations activate if required to switch to another 
configuration before deleting.