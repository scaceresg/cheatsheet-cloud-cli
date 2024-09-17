# Google Cloud Platform

**Contents**:

- [Google Cloud Platform](#google-cloud-platform)
- [Install `gcloud`](#install-gcloud)
  - [Debian/Ubuntu](#debianubuntu)
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
- [Manage `gcloud` CLI Configurations](#manage-gcloud-cli-configurations)
  - [Set a Default Configuration](#set-a-default-configuration)
  - [Create a Configuration](#create-a-configuration)
  - [List Configurations](#list-configurations)
  - [Set Configuration Properties](#set-configuration-properties)
  - [View Configuration Properties](#view-configuration-properties)
  - [Delete a Configuration](#delete-a-configuration)
- [Cloud Storage](#cloud-storage)
  - [Create and Update Buckets](#create-and-update-buckets)
  - [Manage Buckets and Objects](#manage-buckets-and-objects)
  - [Upload/Download Objects](#uploaddownload-objects)
  - [Control Public Access to Data](#control-public-access-to-data)
  - [Access Control Lists (ACLs)](#access-control-lists-acls)
- [Google Kubernetes Engine](#google-kubernetes-engine)
  - [Control Access to GKE Clusters using IAM](#control-access-to-gke-clusters-using-iam)
  - [Create a Standard Cluster](#create-a-standard-cluster)
  - [Create a Deployment](#create-a-deployment)
  - [Scale Up/Down a Deployment](#scale-updown-a-deployment)
  - [Trigger Deployments Rollouts](#trigger-deployments-rollouts)
  - [Trigger Deployments Rollbacks](#trigger-deployments-rollbacks)
  - [Create a LoadBalancer Service](#create-a-loadbalancer-service)
  - [Perform a Canary Deployment](#perform-a-canary-deployment)
  - [Create a Standard Cluster Network Policy](#create-a-standard-cluster-network-policy)
  - [Restrict Incoming Traffic to Pods](#restrict-incoming-traffic-to-pods)
  - [Restrict Outgoing Traffic from the Pods](#restrict-outgoing-traffic-from-the-pods)
  - [Create PersistentVolume and PersistentVolumeClaim](#create-persistentvolume-and-persistentvolumeclaim)
  - [Create StatefulSets](#create-statefulsets)
  - [Define and Use Pod Security Admission](#define-and-use-pod-security-admission)

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

* Use service account impersonation to create a local ADC file:
  
  ```
  gcloud auth application-default login --impersonate-service-account [SERVICE_ACCT_EMAIL]
  ```

  - Credentials are automatically found by the authentication 
  libraries. 

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
  gcloud iam service-accounts add-iam-policy-binding [SERVICE_ACCOUNT_NAME]@[PROJECT_ID].iam.gserviceaccount.com --member="user:[USER_EMAIL]" --role=roles/iam.serviceAccountUser
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
  
  For example:

  ```
  $ CLOUDSDK_CORE_PROJECT=[PROJECT_NAME]
  $ CLOUDSDK_COMPUTE_ZONE=[ZONE_NAME]
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

---
# Cloud Storage

## Create and Update Buckets

* Create buckets with specific settings or advanced configurations:
  
  ```
  gcloud storage buckets create gs://[BUCKET_NAME] --project=[PROJECT_ID] --default-storage-class=[STORAGE_CLASS] --location=[BUCKET_LOCATION] --uniform-bucket-level-access
  ```

  - `BUCKET_NAME`: Must be a unique name
  
  - `BUCKET_LOCATION`: For a configurable dual-region, you must set
  the `--location` flag to be the location code associated with the 
  underlying regions, and you must use the `--placement` flag with 
  a valid pair of regions. Source: https://cloud.google.com/storage/docs/locations

    For example:

    ```
    --location=ASIA --placement=ASIA-EAST1,ASIA-SOUTHEAST1
    ```
  
  - `STORAGE_CLASS`: `STANDARD`, `NEARLINE`, `COLDLINE` or 
  `ARCHIVE`

* Change the storage class of a bucket:

  ```
  gcloud storage buckets update gs://[BUCKET_NAME] --default-storage-class=[STORAGE_CLASS]
  ```

## Manage Buckets and Objects

* List buckets:
  
  ```
  gcloud storage ls
  ```

* Display a bucket's metadata:
  
  ```
  gcloud storage buckets describe gs://[BUCKET_NAME]
  ```

  - Non-editable metadata:
  
    * Bucket name, location, project, metageneration number
  
  - Editable metadata:
  
    * Check https://cloud.google.com/storage/docs/bucket-metadata
  
  - `--format="default(labels)"`: View bucket labels

* Get the current space usage (size) in bytes:
  
  ```
  gcloud storage du gs://[BUCKET_NAME] --summarize
  ```
  ```
  134620      gs://my-bucket
  ```
  
  - The size of the bucket named `my-bucket` is 134,620 bytes

* Add, modify or remove labels:
  
  ```
  gcloud storage buckets update gs://[BUCKET_NAME] --update-labels=[KEY_1]=[VALUE_1]
  ```

  - `--update-labels`: Add a new label or update an existing label
  
  - `--remove-labels`: Remove an existing label
  
  - `--clear-labels`: Remove all labels attached to a bucket

* Move data from one bucket to another:
  
  ```
  gcloud storage cp --recursive gs://[SOURCE_BUCKET]/* gs://[DESTINATION_BUCKET]
  ```

  - `--include-managed-folders`: Copy all your objects and managed folders

* Move or rename an object:
  
  ```
  gcloud storage mv gs://[SOURCE_BUCKET_NAME]/[SOURCE_OBJECT_NAME] gs://[DESTINATION_BUCKET_NAME]/[DESTINATION_OBJECT_NAME]
  ```

* Delete an object:
  
  ```
  gcloud storage rm gs://[BUCKET_NAME]/[OBJECT_NAME]
  ```

* Delete all objects from the source bucket, along with the source 
bucket itself:
  
  ```
  gcloud storage rm --recursive gs://[SOURCE_BUCKET]
  ```

* Delete all objects and managed folders from the source bucket, 
along with the source bucket itself:

  ```
  gcloud alpha storage rm --recursive gs://[SOURCE_BUCKET]
  ```

* Delete the objects and managed folders from the source bucket 
**without** deleting the source bucket itself:

  ```
  gcloud alpha storage rm --all-versions gs://[SOURCE_BUCKET]/**
  ```

## Upload/Download Objects

* Download object:
  
  ```
  gcloud storage cp gs://[BUCKET_NAME]/[OBJECT_NAME] [SAVE_TO_LOCATION]
  ```

* Upload object from a file system:
  
  ```
  gcloud storage cp [OBJECT_LOCATION] gs://[DESTINATION_BUCKET_NAME]
  ```

## Control Public Access to Data

* Make all objects in a bucket publicly readable:
  
  ```
  gcloud storage buckets add-iam-policy-binding gs://[BUCKET_NAME] --member=allUsers --role=roles/storage.objectViewer
  ```

* Make a portion of a bucket publicly readable (**managed folder**):
  
  ```
  gcloud storage managed-folders create gs://[BUCKET_NAME]/[MANAGED_FOLDER_NAME]/
  ```
  
  Or:

  ```
  gcloud storage managed-folders add-iam-policy-binding gs://[BUCKET_NAME]/[MANAGED_FOLDER_NAME] --member=allUsers --role=roles/storage.objectViewer
  ```

* Change **public access prevention**:
  
  - Enable public access prevention:
  
    ```
    gcloud storage buckets update gs://[BUCKET_NAME] --public-access-prevention
    ```

  - Disable public access prevention:
  
    ```
    gcloud storage buckets update gs://[BUCKET_NAME] --no-public-access-prevention
    ```

* View public access prevention status:
  
    ```
    gcloud storage buckets describe gs://[BUCKET_NAME] --format="default(public_access_prevention)"
    ```

## Access Control Lists (ACLs)

* Add or modify an individual grant on an object:
  
  ```
  gcloud storage objects update gs://[BUCKET_NAME]/[OBJECT_NAME] --add-acl-grant=user@gmail.com, role=READER
  ```

* Remove an individual grant on an object:
  
  ```
  gcloud storage objects update gs://[BUCKET_NAME]/[OBJECT_NAME] --remove-acl-grant=user@gmail.com
  ```

* Replace all ACLs for an object:
  
  - Define ACLs in JSON or YAML formats

      ```
      [
        {
          "entity": "project-owners-867489160491",
          "role": "OWNER",
          "projectTeam": {
            "projectNumber": "867489160491",
            "team": "owners"
          },
        },
        {
          "entity": "user-jane@gmail.com",
          "email": "jane@gmail.com",
          "role": "OWNER"
        },
        {
          "entity": "group-gs-announce@googlegroups.com",
          "email": "gs-announce@googlegroups.com",
          "role": "READER"
        }
      ]
      ```

  - Run:
  
    ```
    gcloud storage objects update gs://[BUCKET_NAME]/[OBJECT_NAME] --acl-file=[FILE_LOCATION]
    ```

* Apply a predefined ACL to an object:
  
  - During object upload:
  
    ```
    gcloud storage cp [OBJECT] gs://[BUCKET_NAME] --predefined-acl=[PREDEFINED_ACL]
    ```

    For example:
    
    ```
    gcloud storage cp paris.jpg gs://example-travel-maps --predefined-acl=bucketOwnerRead
    ```

  - On existing buckets or objects:
  
    ```
    gcloud storage objects update gs://[BUCKET_NAME]/[OBJECT_NAME] --predefined-acl=[PREDEFINED_ACL_NAME]
    ```

    For example:

    ```
    gcloud storage objects update gs://example-travel-maps/paris.jpg --predefined-acl=private
    ```

* Get the ACL of an existing resource:
  
  ```
  gcloud storage objects describe gs://[BUCKET_NAME]/[OBJECT_NAME] --format="default(acl)"
  ```

---
# Google Kubernetes Engine

## Control Access to GKE Clusters using IAM

Allow a user with **USERNAME** to create a GKE cluster and deploy 
workloads by **granting permissions to administer all GKE clusters 
and manage resources** inside those clusters in this project.

In Cloud Console:

* Go to IAM & admin > IAM

* Locate the row that corresponds to **USERNAME**, and then click 
on the pencil icon at the right-end of that row to edit that user's 
permissions.

* Click ADD ANOTHER ROLE to add:
  
  - Kubernetes Engine > Kubernetes Engine Cluster Admin

**USERNAME** still lacks some of the rights necessary to deploy a 
cluster. This is because GKE leverages Google Cloud Compute Engine 
instances for the nodes.

**USERNAME** must also be assigned the `iam.serviceAccountUser` 
role on the Compute Engine default service account.

* In Cloud Console, go to IAM & admin > Service accounts

* In the IAM console, click the row that corresponds to the Compute 
Engine default service account to select it.

* Click on the Permissions tab > Grant access

* In the permissions information panel:

  - Type the username for **USERNAME** into the New principals box

  - In the Select a role box, make sure Service Accounts > Service 
  Account User is selected

  - Save

## Create a Standard Cluster

In Cloud Shell:

* Check the active account name:
  
  ```
  gcloud auth list
  ```

* List the Project ID:
  
  ```
  gcloud config list project
  ```

* Set the environment variable for the zone and cluster name:

  ```
  $ export my_region=us-west1
  $ export my_zone=us-west1-a
  $ export my_cluster=standard-cluster-1
  ```

* Configure `kubectl` tab completion in Cloud Shell:
  
  ```
  source <(kubectl completion bash)
  ```

* Create a Kubernetes Standard cluster:

  ```
  gcloud container clusters create $my_cluster \
    --num-nodes 3 [--zone $my_zone | --region $my_region]
  ```

* Configure access to your cluster for the `kubectl` command-line 
tool:

  ```
  gcloud container clusters get-credentials $my_cluster 
    [--zone $my_zone | --region $my_region]
  ```

## Create a Deployment

This Deployment is configured to run three Pod replicas with a 
single `nginx` container in each Pod listening on TCP port 80.

* `nginx-deployment.yaml`
  
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
          - containerPort: 80
  ```

* Create a Deployment using a sample deployment manifest called 
`nginx-deployment.yaml`:

  ```
  kubectl apply -f ./nginx-deployment.yaml
  ```

* View the list of Deployments:
  
  ```
  kubectl get deployments
  ```

* View detailed information about the Deployment:
  
  ```
  gcloud container clusters describe $my_cluster \
    [--zone $my_zone | --region $my_region]
  ```

## Scale Up/Down a Deployment

* Scale down the `nginx-deployment` to 1 Pod replicas:

  ```
  kubectl scale --replicas=1 deployment nginx-deployment
  ```

## Trigger Deployments Rollouts

A deployment's rollout is triggered if and only if the Deployment's 
Pod template (i.e., `.spec.template`) is changed, e.g., if the 
labels or container images of the template are updated.

* Trigger a deployment rollout by updating the version of `nginx` 
in the deployment:

  ```
  kubectl set image deployment.v1.apps/nginx-deployment
    nginx=nginx:1.9.1 
  ```

* Annotate the rollout with details on the change:
  
  ```
  kubectl annotate deployment nginx-deployment \
    kubernetes.io/change-cause="version change to 1.9.1" \
    --overwrite=true
  ```

* View rollout status:
  
  ```
  kubectl rollout status deployment.v1.apps/nginx-deployment
  ```

* View the rollout history of the deployment:
  
  ```
  kubectl rollout history deployment nginx-deployment
  ```

## Trigger Deployments Rollbacks

* Roll back to the previous version of the `nginx` deployment:

  ```
  kubectl rollout undo deployments nginx-deployment
  ```

* View the updated rollout history of the deployment:
  
  ```
  kubectl rollout history deployment nginx-deployment
  ```

* View the details of the latest deployment revision:
  
  ```
  kubectl rollout history deployment/nginx-deployment \
    --revision=3
  ```

## Create a LoadBalancer Service

Services control inbound traffic to an application. The 
LoadBalancer service is configured to distribute inbound traffic 
on TCP port 60000 to port 80 on any containers that have the 
label `app: nginx`.

* `service-nginx.yaml`:
  
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
  spec:
    type: LoadBalancer
    selector:
      app: nginx
    ports:
    - protocol: TCP
      port: 60000
      targetPort: 80
  ```

* Deploy a `LoadBalancer` service type using a manifest file called 
`service-nginx.yaml`:

  ```
  kubectl apply -f ./service-nginx.yaml
  ```

* View the details of the nginx service:
  
  ```
  kubectl get service nginx
  ```

## Perform a Canary Deployment

A Canary Deployment is a separate Deployment used to test a new 
version of your application. A single service targets both the 
canary and normal deployments. And it can direct a subset of users 
to the canary version to mitigate the risk of new releases.

* `nginx-canary.yaml`:
  
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-canary
    labels:
      app: nginx
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
          track: canary
          Version: 1.9.1
      spec:
        containers:
        - name: nginx
          image: nginx:1.9.1
          ports:
          - containerPort: 80
  ```

The manifest for the nginx Service you deployed in the previous 
task uses a label selector to target the Pods with the `app: nginx` 
label. Both the normal Deployment and this new Canary Deployment 
have the `app: nginx` label.

Inbound connections will be distributed by the Service to both the 
normal and canary deployment Pods. The Canary Deployment has fewer 
replicas (Pods) than the normal Deployment, and thus it is available 
to fewer users than the normal deployment.

* Create the canary deployment based on the configuration file:
  
  ```
  kubectl apply -f ./nginx-canary.yaml
  ```

* Verify that both the nginx and the `nginx-canary` deployments are 
present:

  ```
  kubectl get deployments
  ```

## Create a Standard Cluster Network Policy

* Create a cluster network policy to restrict communication between
the Pods. A zero-trust zone is important to prevent lateral attacks 
within the cluster when an intruder compromises one of the Pods.

  ```
  gcloud container clusters create $my_cluster \
    --num-nodes 3 --enable-ip-alias --zone $my_zone \
    --enable-network-policy
  ```

* Configure access to your cluster for the `kubectl` command-line 
tool:

  ```
  gcloud container clusters get-credentials $my_cluster \
    --zone $my_zone
  ```

* Run a simple web server application with the label `app=hello`, 
and expose the web application internally in the cluster:

  ```
  kubectl run hello-web --labels app=hello \
    --image=gcr.io/google-samples/hello-app:1.0 \
    --port 8080 --expose
  ```

## Restrict Incoming Traffic to Pods

Create a sample NetworkPolicy by defining an ingress policy that 
allows access to Pods labeled `app: hello` from Pods labeled 
`app: foo`

* `hello-allow-from-foo.yaml`:
  
  ```
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: hello-allow-from-foo
  spec:
    policyTypes:
    - Ingress
    podSelector:
      matchLabels:
        app: hello
    ingress:
    - from:
      - podSelector:
          matchLabels:
            app: foo
  ```

* Create the ingress policy:
  
  ```
  kubectl apply -f hello-allow-from-foo.yaml
  ```

* Verify that the policy was created:
  
  ```
  kubectl get networkpolicy
  ```
  ```
  NAME                   POD-SELECTOR   AGE
  hello-allow-from-foo   app=hello      7s
  ```

* Validate the ingress policy by running a temporary Pod called 
`test-1` with the label `app=foo` and get a shell in the Pod:

  ```
  kubectl run test-1 --labels app=foo --image=alpine \
    --restart=Never --rm --stdin --tty
  ```
 
  - `--stdin` (alternatively `-i`): creates an interactive session 
  attached to STDIN on the container.

  - `--tty` (alternatively `-t`): allocates a TTY for each container 
  in the pod.

  - `--rm`: instructs Kubernetes to treat this as a temporary Pod 
  that will be removed as soon as it completes its startup task. As 
  this is an interactive session it will be removed as   soon as the 
  user exits the session.

  - `--label` (alternatively `-l`): adds a set of labels to the pod.

  - `--restart`: defines the restart policy for the Pod.
  
* Make a request to the `hello-web:8080` endpoint to verify that the 
incoming traffic is allowed:

  ```
  # wget -qO- --timeout=2 http://hello-web:8080
  ```
  ```
  If you don't see a command prompt, try pressing enter.
  / # wget -qO- --timeout=2 http://hello-web:8080
  Hello, world!
  Version: 1.0.0
  Hostname: hello-web-8b44b849-k96lh
  / #
  ```

* Now, run a different Pod using the same Pod name but using a label, 
`app=other`, that does not match the podSelector in the active 
network policy. This Pod should not have the ability to access the 
`hello-web` application.

  ```
  kubectl run test-1 --labels app=other --image=alpine \
    --restart=Never --rm --stdin --tty
  ```

* Make a request to the hello-web:8080 endpoint to verify that the 
incoming traffic is not allowed:

  ```
  # wget -qO- --timeout=2 http://hello-web:8080
  ```
  ```
  If you don't see a command prompt, try pressing enter.
  / # wget -qO- --timeout=2 http://hello-web:8080
  wget: download timed out
  / #
  ```

## Restrict Outgoing Traffic from the Pods

You can restrict outgoing (egress) traffic as you do incoming 
traffic. However, in order to query internal hostnames (such as 
`hello-web`) or external hostnames (such as `www.example.com`), you 
must allow DNS resolution in your egress network policies. DNS 
traffic occurs on port 53, using TCP and UDP protocols.

Create a NetworkPolicy manifest file `foo-allow-to-hello.yaml`. This 
file will define:

* A policy that permits Pods with the label `app: foo` to communicate 
with Pods labeled `app: hello` on any port number, and

* Also allows the Pods labeled `app: foo` to communicate to any 
computer on UDP port 53, which is used for DNS resolution. Without 
the DNS port open, you will not be able to resolve the hostnames.

* `foo-allow-to-hello.yaml`:

  ```
  kind: NetworkPolicy
  apiVersion: networking.k8s.io/v1
  metadata:
    name: foo-allow-to-hello
  spec:
    policyTypes:
    - Egress
    podSelector:
      matchLabels:
        app: foo
    egress:
    - to:
      - podSelector:
          matchLabels:
            app: hello
    - to:
      ports:
      - protocol: UDP
        port: 53
  ```

* Create the egress policy:
  
  ```
  kubectl apply -f foo-allow-to-hello.yaml
  ```

* Verify that the policy was created:
  
  ```
  kubectl get networkpolicy
  ```
  ```
  NAME                   POD-SELECTOR   AGE
  foo-allow-to-hello     app=foo        7s
  hello-allow-from-foo   app=hello      5m
  ```

* Validate the egress policy by running a new web application called 
`hello-web-2` and expose it internally in the cluster:
  
  ```
  kubectl run hello-web-2 --labels app=hello-2 \
    --image=gcr.io/google-samples/hello-app:1.0 \
    --port 8080 --expose
  ```

* Again, create a temporary Pod with label `app=foo`:
  
  - Test connection to http://hello-web:8080 $\rightarrow$ CAN 
  establish connection because of label `app: hello`

  - Test connection to http://hello-web-2:8080 $\rightarrow$ CANNOT 
  establish connection because of label `app=hello-2`

  - Test connection to http://www.example.com $\rightarrow$ CANNOT 
  establish connection because the network policies do not allow 
  external http traffic (tcp port 80)
  
## Create PersistentVolume and PersistentVolumeClaim

You don't need to directly configure PV objects or create Compute 
Engine persistent disks. Instead, you can create a PVC, and 
Kubernetes automatically provisions a persistent disk for you.

* Create a 30 gigabyte PVC called `hello-web-disk` that can be 
mounted as a read-write volume on a single node at a time.

* `pvc-demo.yaml`:
  
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: hello-web-disk
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 30Gi
  ```

* Show that you currently have no PVCs:
  
  ```
  kubectl get persistentvolumeclaim
  ```
  ```
  No resources found in default namespace.
  ```

* Create the PVC:
  
  ```
  kubectl apply -f pvc-demo.yaml
  ```

* Show list of PVCs:
  
  ```
  kubectl get persistentvolumeclaim
  ```
  ```
  NAME           STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGE CLASS   AGE
  hello-web-disk Pending                                          standard-rwo    15s
  ```

Mount the PVC to a Pod and attach the `pvc-demo-volume` to the Pod 
and mount that volume to the path `/var/www/html` inside the nginx 
container.

Files saved to this directory inside the container will be saved to 
the persistent volume and persist even if the Pod and the container 
are shutdown and recreated.

* `pod-volume-demo.yaml`:
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: pvc-demo-pod
  spec:
    containers:
      - name: frontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
          name: pvc-demo-volume
    volumes:
      - name: pvc-demo-volume
        persistentVolumeClaim:
          claimName: hello-web-disk
  ```

* Create the Pod with the volume:
  
  ```
  kubectl apply -f pod-volume-demo.yaml
  ```

* List the Pods in the cluster:
  
  ```
  kubectl get pods
  ```
  
  - The volume is mounted before the status changes to "Running".

* Verify the PVC is accessible within the Pod, you must gain shell 
access to your Pod. To start the shell session, execute the following 
command:

  ```
  kubectl exec -it pvc-demo-pod -- sh
  ```

* Create a simple text message as a web page in the Pod enter the 
following commands:

  ```
  # echo Test webpage in a persistent volume!>/var/www/html/index.html
  # chmod +x /var/www/html/index.html
  ```

* Verify the text file contains your message and then exit:
  
  ```
  # cat /var/www/html/index.html
  ```
  ```
  Test webpage in a persistent volume!
  ```

* Test the persistency of the PVC by deleting the Pod. Show your PVC:

  ```
  kubectl get persistentvolumeclaim
  ```
  ```
  NAME           STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGE CLASS   AGE
  hello-web-disk Bound     pvc-8...34   30Gi       RWO            standard-rwo    22m
  ```

* Redeploy and access the Pod again to verify that the text file 
still contains your message.

## Create StatefulSets

A StatefulSet is like a Deployment, except that the Pods are given 
unique identifiers.

Use your PVC from the previous section to create a StatefulSet. 
Before you can use the PVC with the StatefulSet, you must delete 
the Pod that is currently using it.

* Create a manifest file `statefulset-demo.yaml` that creates a 
StatefulSet:

  - The manifest includes a LoadBalancer Service and 3 replicas of 
  a Pod containing an `nginx` container 
  
  - It also contains a `volumeClaimTemplate` for 30 gigabyte PVCs 
  with the name `hello-web-disk`. 
  
  - The nginx containers mount the PVC called `hello-web-disk` at 
  `/var/www/html`:

* `statefulset-demo.yaml`:
  
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: statefulset-demo-service
  spec:
    ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
    type: LoadBalancer
  ---

  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: statefulset-demo
  spec:
    selector:
      matchLabels:
        app: MyApp
    serviceName: statefulset-demo-service
    replicas: 3
    updateStrategy:
      type: RollingUpdate
    template:
      metadata:
        labels:
          app: MyApp
      spec:
        containers:
        - name: stateful-set-container
          image: nginx
          ports:
          - containerPort: 80
            name: http
          volumeMounts:
          - name: hello-web-disk
            mountPath: "/var/www/html"
    volumeClaimTemplates:
    - metadata:
        name: hello-web-disk
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 30Gi
  ```

* Create the StatefulSet with the volume:
  
  ```
  kubectl apply -f statefulset-demo.yaml
  ```
  ```
  service "statefulset-demo-service" created
  statefulset.apps "statefulset-demo" created
  ```

You now have a StatefulSet running behind a service named 
`statefulset-demo-service`!

* View the StatefulSet details:
  
  ```
  kubectl describe statefulset statefulset-demo
  ```
  ```
  Normal  SuccessfulCreate  10s   statefulset-controller
  Message: create Claim hello-web-disk-statefulset-demo-0 Pod statefulset-demo-0 in StatefulSet statefulset-demo success

  Normal  SuccessfulCreate  10s   statefulset-controller
  Message: create Pod statefulset-demo-0 in StatefulSet statefulset-demo successful
  ```

* List the Pods in the cluster:
  
  ```
  kubectl get pods
  ```
  ```
  NAME                 READY     STATUS    RESTARTS   AGE
  statefulset-demo-0   1/1       Running   0          6m
  statefulset-demo-1   1/1       Running   0          3m
  statefulset-demo-2   1/1       Running   0          2m
  ```

* List the PVCs:
  
  ```
  kubectl get pvc
  ```
  ```
  NAME                          STATUS    VOLUME           CAPACITY ACCESS
  hello-web-disk                Bound     pvc-86117ea6-...34   30Gi    RWO
  hello-web-disk-st...-demo-0   Bound     pvc-92d21d0f-...34   30Gi    RWO
  hello-web-disk-st...-demo-1   Bound     pvc-9bc84d92-...34   30Gi    RWO
  hello-web-disk-st...-demo-2   Bound     pvc-a526ecdf-...34   30Gi    RWO
  ```

The original hello-web-disk is still there and you can now see the 
individual PVCs that were created for each Pod in the new 
statefulset Pod.

* View the details of the first PVC in the StatefulSet:
  
  ```
  kubectl describe pvc hello-web-disk-statefulset-demo-0
  ```

* Verify the connection of Pods in StatefulSets to particular PVs 
as the Pods are stopped and restarted:

  - Gain shell access to your Pod to start the shell session
    
    ```
    kubectl exec -it statefulset-demo-0 -- sh
    ```
  
  - Verify that there is no `index.html` text file in the 
  `/var/www/html` directory

  - Create a simple text message as a web page in the Pod:
    
    ```
    # echo Test webpage in a persistent volume!>/var/www/html/index.html
    # chmod +x /var/www/html/index.html
    ```

  - Delete the Pod where you updated the file on the PVC:
    
    ```
    kubectl delete pod statefulset-demo-0
    ```

  - You will see that the StatefulSet is automatically restarting 
  the `statefulset-demo-0` Pod. Connect again to the shell on the 
  new `statefulset-demo-0` Pod and verify the `index.html` file 
  still contains your message.

## Define and Use Pod Security Admission

PodSecurity is a Kubernetes admission controller that lets you 
apply **Pod Security Standards** to Pods running on your GKE 
clusters. Pod Security Standards are **predefined security policies 
that cover the high-level needs of Pod security** in Kubernetes. 
These policies range from being highly permissive to highly 
restrictive.

Create a Pod Security Policy to create **unprivileged Pods for
specific namespaces** in the cluster:

* Unprivileged Pods do not allow users to execute code as root, and 
have limited access to devices on the host.

* Create environment variables for the Google Cloud zone and cluster 
name:

  ```
  $ export my_zone=ZONE
  $ export my_cluster=standard-cluster-1
  ```

* Configure tab completion for the kubectl command-line tool:
  
  ```
  source <(kubectl completion bash)
  ```

* Configure access to your cluster for kubectl:
  
  ```
  gcloud container clusters get-credentials $my_cluster \
    --zone $my_zone
  ```

* Create namespaces in your cluster:

  - `baseline-ns`: For permissive workloads
  
  - `restricted-ns`: For highly restricted workloads
  
  ```
  $ kubectl create ns baseline-ns
  $ kubectl create ns restricted-ns
  ```

* Apply the following Pod Security Standards:
  
  - Workloads in the `baseline-ns` namespace that violate the 
  baseline policy are allowed, and the client displays a warning 
  message:
  
    * **baseline**: Apply to `baseline-ns` in the **warn** mode
  
  - Workloads in the `restricted-ns` namespace that violate the 
  restricted policy are rejected, and GKE adds an entry to the 
  audit logs:

    * **restricted**: Apply to `restricted-ns` in the **enforce** 
    mode
  
  ```
  $ kubectl label --overwrite ns baseline-ns \
    pod-security.kubernetes.io/warn=baseline
  $ kubectl label --overwrite ns restricted-ns \
    pod-security.kubernetes.io/enforce=restricted
  ```

* Verify that the labels were added:
  
  ```
  kubectl get ns --show-labels
  ```
  ```
  baseline-ns       Active   74s   kubernetes.io/metadata.name=baseline-ns,pod-security.kubernetes.io/warn=baseline
  restricted-ns     Active   18s   kubernetes.io/metadata.name=restricted-ns,pod-security.kubernetes.io/enforce=restricted
  default           Active   57m   kubernetes.io/metadata.name=default
  kube-public       Active   57m   kubernetes.io/metadata.name=kube-public
  kube-system       Active   57m   kubernetes.io/metadata.name=kube-system
  ```

Verify the PodSecurity admission controller is working by deploying 
a Pod running a container with priviliged escalation 
`securityContext: privileged: true` to both namespaces `baseline-ns` 
and `restricted-ns`:

* `psa-workload.yaml`:
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
      securityContext:
        privileged: true
  ```

* Apply to the `baseline-ns` namespace. It's created with a warning 
message:
  
  ```
  kubectl apply -f psa-workload.yaml --namespace=baseline-ns
  ```
  ```
  Warning: would violate PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
  pod/nginx created
  ```

* Apply to the `restricted-ns` namespace: Pod won't be created:
  
  ```
  kubectl apply -f psa-workload.yaml --namespace=restricted-ns
  ```
  ```
  Error from server (Forbidden): error when creating "workload.yaml": pods "nginx"
  is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation
  != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false),
  unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]),
  runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true),
  seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type
  to "RuntimeDefault" or "Localhost")
  ```

Create a ClusterRole that can be used in a ClusterRoleBinding to 
**tie the policy to accounts** that require to deploy pods with 
unprivileged access:
  
  - Used in ClusterRolebinding that ties the policy to accounts 
  that require the ability to deploy pods with unprivileged access.

