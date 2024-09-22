# Google Compute Engine

**Contents**:

- [Google Compute Engine](#google-compute-engine)
- [View/Set Region/Zone and Describe Project](#viewset-regionzone-and-describe-project)
- [Work with VMs](#work-with-vms)


# View/Set Region/Zone and Describe Project

* View your region and zone details:
  
  - Region:
  
    ```
    gcloud config get-value compute/region
    ```

  - Zone:
  
    ```
    gcloud config get-value compute/zone
    ```

* View a list of available regions and zones:

  ```
  gcloud compute regions list
  ```

  ```
  gcloud compute zones list
  ```

* Set a default region and zone in the metadata server:
  
  ```
  gcloud compute project-info add-metadata \
   --metadata google-compute-default-region=[REGION],google-compute-default-zone=[ZONE]
  ```

  For example:
  
  ```
  gcloud compute project-info add-metadata \
   --metadata google-compute-default-region=europe-west1,google-compute-default-zone=europe-west1-b
  ```

* Unset metadata:
  
  ```
  gcloud compute project-info remove-metadata \
   --keys=google-compute-default-region,google-compute-default-zone
  ```

* Query information about your Compute Engine project, such as 
project metadata, ssh keys, and quota metrics:

  ```
  gcloud compute project-info describe
  ```

* Set the `PROJECT_ID` environment variable from the configuration
file:
  
  ```
  export PROJECT_ID="$(gcloud config get-value project -q)"
  ```


# Work with VMs

* Create a VM:
  
  ```
  gcloud compute instances create [VM_NAME] \
    [--image [IMAGE] | --image-family [IMAGE_FAMILY]] \
    --image-project [IMAGE_PROJECT]
  ```

* List VMs in a project:
  
  ```
  gcloud compute instances list
  ```
