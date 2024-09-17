# Google Cloud Storage

**Contents**:

- [Google Cloud Storage](#google-cloud-storage)
  - [Create and Update Buckets](#create-and-update-buckets)
  - [Manage Buckets and Objects](#manage-buckets-and-objects)
  - [Upload/Download Objects](#uploaddownload-objects)
  - [Control Public Access to Data](#control-public-access-to-data)
  - [Access Control Lists (ACLs)](#access-control-lists-acls)

---

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