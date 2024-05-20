# Managing IAC with Terraform, Cloud Build, and GitOps

This tutorial explains how to manage infrastructure as code with Terraform and Cloud Build using the popular GitOps methodology. GitOps uses a Git repository to store the desired environment state. Terraform enables predictable creation, change, and improvement of cloud infrastructure using code. Cloud Build, a Google Cloud continuous integration service, automatically applies Terraform manifests to your environment.

This tutorial is for developers and operators looking for an elegant strategy to predictably make changes to infrastructure. Familiarity with Google Cloud, Linux, and GitHub is assumed.

## Capabilities
This tutorial will help you with:
- Version control
- Continuous integration
- Continuous delivery
- Continuous testing

## Architecture
This tutorial uses GitHub branches—`dev` and `prod`—to represent environments. These environments are defined by Virtual Private Cloud (VPC) networks within a Google Cloud project.

The process starts when you push Terraform code to either the `dev` or `prod` branch. Cloud Build triggers and then applies Terraform manifests to achieve the desired state in the respective environment. For other branches, Cloud Build runs `terraform plan` without applying any changes.

## Objectives
1. Set up your GitHub repository.
2. Configure Terraform to store state in a Cloud Storage bucket.
3. Grant permissions to your Cloud Build service account.
4. Connect Cloud Build to your GitHub repository.
5. Change your environment configuration in a feature branch.
6. Promote changes to the development environment.
7. Promote changes to the production environment.

## Costs
This tutorial uses the following billable components of Google Cloud:
- Cloud Build
- Cloud Storage
- Compute Engine

## Prerequisites
- Create a Google Cloud account and project.
- Ensure billing is enabled.
- Activate Cloud Shell.

## Steps

### 1. Setting Up Your GitHub Repository
1. Fork the [solutions-terraform-cloudbuild-gitops](https://github.com/GoogleCloudPlatform/solutions-terraform-cloudbuild-gitops.git) repository.
2. Clone the forked repository:
    ```bash
    cd ~
    git clone https://github.com/YOUR_GITHUB_USERNAME/solutions-terraform-cloudbuild-gitops.git
    cd ~/solutions-terraform-cloudbuild-gitops
    ```

### 2. Configuring Terraform to Store State in a Cloud Storage Bucket
1. Create a Cloud Storage bucket:
    ```bash
    PROJECT_ID=$(gcloud config get-value project)
    gsutil mb gs://${PROJECT_ID}-tfstate
    gsutil versioning set on gs://${PROJECT_ID}-tfstate
    ```
2. Replace the `PROJECT_ID` placeholder in `terraform.tfvars` and `backend.tf` files:
    ```bash
    cd ~/solutions-terraform-cloudbuild-gitops
    sed -i s/PROJECT_ID/$PROJECT_ID/g environments/*/terraform.tfvars
    sed -i s/PROJECT_ID/$PROJECT_ID/g environments/*/backend.tf
    ```

### 3. Granting Permissions to Your Cloud Build Service Account
1. Retrieve the email for your project's Cloud Build service account:
    ```bash
    CLOUDBUILD_SA="$(gcloud projects describe $PROJECT_ID --format 'value(projectNumber)')@cloudbuild.gserviceaccount.com"
    ```
2. Grant the required access:
    ```bash
    gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:$CLOUDBUILD_SA --role roles/editor
    ```

### 4. Connecting Cloud Build to Your GitHub Repository
1. Install the Cloud Build GitHub app and configure it to connect your repository.

### 5. Changing Your Environment Configuration in a Feature Branch
1. Create a new branch and fix a typo in the `modules/firewall/main.tf` file.
2. Create a pull request and review the terraform plan.

### 6. Promoting Changes to the Development Environment
1. Merge the pull request into the `dev` branch.
2. Verify the deployment and the terraform state in Cloud Storage.

### 7. Promoting Changes to the Production Environment
1. Create a pull request to merge the `dev` branch into the `prod` branch.
2. Review the changes and merge the pull request.
3. Verify the deployment and the terraform state in Cloud Storage.

## Cleanup
To avoid continued billing, delete the resources you created.

### Deleting the Project
1. Go to the [Manage resources](https://console.cloud.google.com/iam-admin/projects) page.
2. Select your project and click **Delete**.

### Deleting the GitHub Repository
1. Navigate to the main page of your forked repository.
2. Go to **Settings** > **Branches** and delete branch protection rules.
3. Optionally, uninstall the Cloud Build app from GitHub.
4. Delete the repository from GitHub.

## Additional Information
- Add deployments for separate use cases.
- Create additional environments.
- Use a project per environment.

For a detailed guide, refer to the complete [tutorial](https://cloud.google.com/build/docs/deploying-builds/deploying-to-gce-terraform).
