# GAN Training Documentation

Documenting an automated process for training and monitoring GAN models in AWS, Azure and GCP.

The first step is to setup access for the training and monitoring scripts running on their VMs to read and write to blob storage. Then you'll need to clone the Infrastructure as Code (IaC) repository for your chosen cloud provider.

## Table of Contents

* [Dependencies](#dependencies)
  * [Training and Monitor Repositories](#training-and-monitor-repositories)
* [AWS](#aws)
  * [Local Access Setup](#aws-local-access-setup)
  * [Cloud Storage Access Setup](#aws-cloud-storage-access-setup)
  * [Training](#aws-training)
  * [Training VM Teardown](#aws-training-teardown)
* [Azure](#azure) (WIP)
  * [Local Access Setup](#azure-local-access-setup)
  * [Cloud Storage Access Setup](#azure-cloud-storage-access-setup)

## Dependencies

* Terraform==1.0.10
* AWS
  * [AWS CLI](https://aws.amazon.com/cli/)
  * [AWS IaC repository](https://github.com/sem-onyalo/gan-training-iac-aws-terraform)
* Azure (WIP)
  * [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
  * [Azure IaC repository](https://github.com/sem-onyalo/gan-training-iac-azure-terraform)
* GCP (WIP)
  * [GCP IaC repository](https://github.com/sem-onyalo/gan-training-iac-gcp-terraform)

### Training and Monitor Repositories

* [Model training repository](https://github.com/sem-onyalo/gan-training-model)
* [Model training monitor repository](https://github.com/sem-onyalo/gan-training-monitor-flask)

## AWS

### AWS Local Access Setup

If you have never setup local access to your AWS account via the CLI then run the command below after you [install the CLI](https://aws.amazon.com/cli/).

```
aws configure
```

### AWS Cloud Storage Access Setup

1. Navigate to the IAM service page.

    ![AWS IAM service search](imgs/aws/setup/1-iam.png)

2. Click on the `Roles` link.

    ![AWS Roles](imgs/aws/setup/2-roles.png)

3. Click on the `Create role` button.

    ![AWS create role](imgs/aws/setup/3-create-role.png)

4. Select the `EC2` use case link and then click on the `Next: Permissions` button.

    ![AWS EC2 use case role](imgs/aws/setup/4-ec2-use-case.png)

5. Type `S3` in the search bar, select the checkbox for the `AmazonS3FullAccess` policy and then click on the `Next: Tags` button.

    ![AWS role policy](imgs/aws/setup/5-attach-policy.png)

6. You can optionally add tags and then click the `Next: Review` button.

    ![AWS role tags](imgs/aws/setup/6-role-tags.png)

7. Enter the text "gan-training-storage-access" into the `Role name` text box and then click the `Create role` button.

    ![AWS create role final](imgs/aws/setup/7-create-role.png)

### AWS Training

1. Open a new terminal, `cd` to your local code folder and clone the AWS IaC repository.

    ```
    cd ~/code
    
    git clone https://github.com/sem-onyalo/gan-training-iac-aws-terraform.git

    cd gan-training-iac-aws-terraform
    ```

2. Execute the `./run` bash script contained in the repository with the specified AWS specific commands.

    ```
    ./run -c 'python3 main.py --train -c aws_s3' -f 'python3 app.py -s aws_s3'
    ```

3. After the script finishes executing it will output the IP address of the VM hosting the monitoring page. Use that IP and port `8080` to monitor the training (for example, http://35.88.174.24:8080). Note that data on the webpage will not show up right away because the training VM needs time to spin up, install the required libraries and initialize the GPU for training.

    ![Monitoring server IP address output](imgs/aws/train/1-monitoring-server-ip.png)

3. You can use the `Evaluation epochs` dropdown to view the loss/accuracy plots, the generated images and the mini-batch training metrics for the selected evaluation epoch. This should help you to pick the model that generates the best images. Models for each evaluation epoch are stored in an S3 bucket called `gan-training-storage`.

    ![AWS monitor page](imgs/aws/train/2-monitor-page.png)

    ![AWS storage container](imgs/aws/train/3-storage-container.png)

### AWS Training Teardown

**IMPORTANT:** Remember to destroy the training VM when you have completed training and/or have downloaded your desired model, as it [costs money to keep up and running](https://aws.amazon.com/ec2/instance-types/p3/). You can destroy the resources with the following command.

```
terraform destroy -auto-approve
```

## Azure

### Azure Local Access Setup

If you have never setup local access to your AWS account via the CLI then run the command below after you [install the CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

```
az login
```

### Azure Cloud Storage Access Setup

1. Navigate to Azure Active Directory

    ![Azure AAD service search](imgs/azure/setup/1-aad.png)

2. Click on the `App registrations` link.

    ![Azure app registrations](imgs/azure/setup/2-app-registrations.png)

3. Client on the `New registration` link.

    ![Azure new app registration](imgs/azure/setup/3-new-registration.png)

4. Enter the text "gan-training-application" into the `Name` text box.

    ![Azure app registration name](imgs/azure/setup/4-application-name.png)

5. Specify which account types you want to grant access to the storage container. Shown below is the default option.

    ![Azure app registration supported account type](imgs/azure/setup/5-supported-account-type.png)

6. Click on the `Register` button.

    ![Azure app register](imgs/azure/setup/6-register.png)
