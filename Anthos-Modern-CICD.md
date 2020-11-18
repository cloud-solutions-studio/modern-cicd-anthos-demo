# Anthos Platform Demo

## Disclaimer

**Please ensure you create and select a new project in the next step to ensure this tutorial functions properly**

Googlers: You should deploy the new project within the "untrusted/experimental-untrusted/experimental-gke" folder in the "google.com" org to avoid issues. You may need to accept the T&C [here](http://go/experimental-folder-access) to be granted access to this folder.

Click Start to Begin

## Project Selection
<walkthrough-project-billing-setup></walkthrough-project-billing-setup>

## Overview

This tutorial will take you through the steps required to setup a GCP project and deploy the "Modern CI/CD with Anthos" demo environment. 


It will take  approximately **5 minutes** of manual setup and approximately **30 minutes** of automated deployment.

## Prerequisites
**Download Repository & Setup**

**Step 1.)** Run the command below in your Cloud Shell to enter your home directory.
```bash
cd ~
```


**Step 2.)** Clone the Github repository to your Cloud Shell.
```bash
git clone https://github.com/GoogleCloudPlatform/solutions-modern-cicd-anthos.git
```


**Step 3.)** Enter the "solutions-modern-cicd-anthos" directory.
```bash
 cd solutions-modern-cicd-anthos
```


Click **Next** when finished to continue.

## Enable APIs & Set Env Variables

**Step 1.)** Set the us-central1 region to deploy infrastructure.
```bash
export REGION="us-central1"
gcloud config set compute/region ${REGION}
```


**Step 2.)** Set your project.
```bash
export PROJECT_ID={{project-id}}
gcloud config set core/project ${PROJECT_ID}
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format 'value(projectNumber)')
```


**Step 3.)** Enable necessary APIs on the project.
```bash
gcloud services enable cloudbuild.googleapis.com
gcloud services enable anthos.googleapis.com
gcloud services enable serviceusage.googleapis.com
gcloud services enable cloudkms.googleapis.com
gcloud services enable containeranalysis.googleapis.com
```


**NOTE:** Enabling the APIs will take a few minutes. If you encounter an error running the subsequent steps, wait a few minutes until the APIs are enabled and try again.


**Step 4.)** Grant the Cloud Build Service Account IAM permissions.
```bash
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com --role roles/owner
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com --role roles/containeranalysis.admin
```


Click **Next** when finished to continue.

## Cloud Build Deployment

**Step 1.)** Submit Cloud Build job to begin automated deployment of the environment. **This takes around 30 minutes.**
```bash
gcloud builds submit --substitutions=_PROJECT_ID=${PROJECT_ID}
```


**NOTE:** If the Cloud Build job fails, re-run the "gcloud build" command. The command will take substantially less time to run and will pick up where it last hung. If this still does not work, skip to Step 8 of this tutorial to clean up the environment and start again.


**Step 2.)** Take note of Gitlab URL/credentials that are provided when the Cloud Build job completes.


Cloud Build job output:
```
Log in to your GitLab instance at: https://gitlab.{{project-id}}.demo.anthos-platform.dev
Step #4 - "setup-gitlab": + echo -e '\e[32mUsername: root'
Step #4 - "setup-gitlab": Username: root
Step #4 - "setup-gitlab": + echo -e '\e[32mPassword: YOURPASSWORD
```


URL for Gitlab:
```bash
echo "https://gitlab.endpoints.${PROJECT_ID}.cloud.goog"
```


User and Password for GitLab are stored in Secrets Manager:
```bash
export GITLAB_USER=$(gcloud secrets versions access latest --secret="gitlab-user")
export GITLAB_PASSWORD=$(gcloud secrets versions access latest --secret="gitlab-password")
```

```bash
echo "User: ${GITLAB_USER}"
echo "Password: ${GITLAB_PASSWORD}"
```


Click **Next** when finished to continue.

## Environment Overview

For this demo, the Anthos platform consists of **5 GKE clusters**: 

* **Two production clusters**, prod-us-central1 in us-central1 region and prod-us-east1 in us-east1 region.

* **One staging cluster**, staging-us-central1 staging cluster in us-central1 region.

* **One dev cluster**, dev-us-central1 dev cluster in us-central1

* **One cluster dedicated to GitLab (self-managed)**


Click **Next** when finished to continue.

## Congratulations!

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

You've successfully completed the deployment setup for the "Modern CICD with Anthos" demo. Please navigate to slide 2 of the slide deck located [here](https://docs.google.com/presentation/d/1p4jyKluC1oCcQ1HvwLOjhpKkckQ4CKD6rXxIyXJXDhg/edit#slide=id.g7f25a49472_0_7).


For more detailed instructions on how a customer would interact with this Modern CI/CD infrastructure (onboarding a new application, adding a new cluster, developer workflow, etc.) please see the [User Guide](https://github.com/GoogleCloudPlatform/solutions-modern-cicd-anthos/blob/master/docs/index.md).


Click **Next** to clean up your environment.

## Environment Clean Up

**Step 1.)** Remove infrastructure created from Cloud Build automated deployment.
```bash
gcloud builds submit --substitutions=_PROJECT_ID=${PROJECT_ID} --config cloudbuild-destroy.yaml
```


**Step 2.)** Unset variables (optional)
```bash
unset PROJECT_ID
unset REGION
```
