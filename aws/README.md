<!--
# Copyright 2021 VMware, Inc
# SPDX-License-Identifier: BSD-2-Clause
-->
# Purpose

This [Terraform](https://terraform.io) project is designed to build an architecture in AWS that corresponds to the [TKO Reference Design on AWS](https://docs.vmware.com/en/VMware-Tanzu/services/tanzu-reference-architecture/GUID-reference-designs-tko-on-aws.html)

Specifically, this automation will build:
- a management VPC
- a workload VPC
- transit gateway and associated networking
- a jump box from which to execute the commands necessary to install TKG

This Terraform relies on the Tanzu CLI CloudFormation stack to be pre-created in the AWS account it is executing against, as it will utilize the IAM permissions delegated to the jump box. 

**Note**: If this AWS account has never had the Tanzu CloudFormation stack configured, you should run `tanzu management-cluster permissions aws set` before executing these Terraform manifests.

## Getting Started

### Prepare the Environment

First, be sure that your AWS access credentials are available within your environment.
 
```bash
export AWS_ACCESS_KEY_ID=<your AWS access key>
export AWS_SECRET_ACCESS_KEY=<your AWS secret access key>
export AWS_REGION=us-east-1  # ensure the region is set correctly. this must agree with what you set in the tf files below.
```

### Prepare Terraform

* First, initialize Terraform by executing `terraform init`
* Next, set required variables in `terraform.tfvars`
  * `aws_region` Should be set to your AWS region
  * `jb_key_file` Should be set to your jumpbox key e.g. `~/tkgkp.pem` if you need to create key pair you can use the aws cli like:
       `aws ec2 create-key-pair --key-name tkg-kp --query 'KeyMaterial' --output text > ~/tkgkp.pem`
  * `jb_key_pair` Should be set to your jumpbox key pair name in AWS
  * `run_id` [Optional] Can be set to a value that will be propagated as a tag to all resources created by this terraform. Primarily useful for running this in a CI/CD pipeline.

###  



# Clean up

In some instances it is possible that `terraform destroy` will not be able to clean up after a failed Tanzu install. Especially in situations where the management cluster only comes partially up for whatever reason. In this circumstance you can recursively delete the VPCs that failed to get destroyed and use the tags "CreatedBy: Arcas" to find anything that was generated by this terraform. Additionally, you can set the RunId input variable to help you track things created by a specific run of this terraform.

# Customizing

This repository is designed to be customizable -- especially at the root `main.tf` level. For example, if you need to reference an existing transit gateway, it is expected you can remove the resource block for the transit gateway, and replace the transit gateway ids in with an input variable. If you need more VPCs, you can add them to the `main.tf` file.

# Tanzu Package configurations 

You can add Tanzu packages configurations as needed into templates files to make them functional. Otherwise they will installed with default values as defined into template files. 
  ## Pinniped 
  Add following external Identity provider(IDP) details into cluster config file(**tkg_vpc/templates/mgmt.tpl**) to integrate Pinniped with external IDP.

```yaml
    * OIDC_IDENTITY_PROVIDER_CLIENT_ID: ""
    * OIDC_IDENTITY_PROVIDER_CLIENT_SECRET: ""
    * OIDC_IDENTITY_PROVIDER_ISSUER_URL: ""
    * OIDC_IDENTITY_PROVIDER_NAME: ""
```

## Prometheus

To setup secured prometheus monitoring update the following entries in the prometheus-data-values.yaml file

  ```
    * ingress.virtual_host_fqdn: 
    * tlsCertificate.tls.crt:
    * tlsCertificate.tls.key:
    * tlsCertificate.ca.crt:
  ```

## Grafana

To setup secured grafana dashboard update the following entries in the grafana-data-values.yaml file

  ```
    * ingress.virtual_host_fqdn: 
    * tlsCertificate.tls.crt:
    * tlsCertificate.tls.key:
    * tlsCertificate.ca.crt:
  ```

## Harbor   

To set your own passwords and secrets, update the following entries in the harbor-data-values.yaml file
```
 * hostname 
 * harborAdminPassword
 * secretKey
 * database.password
 * core.secret
 * core.xsrfKey
 * jobservice.secret
 * registry.secret
```
# Terraform Lifecycle 
## terraform plan command will verify that the .tf files are good to go.
`$ terraform plan`

## Assuming the previous command output looks good, run terraform
`$ terrform apply`

## Execute following tasks after jumpbox created 
  
```bash 
# Login to jumpbox machine
$ ssh -i "path/<keypair file name>.pem" ubuntu@<jumpbox ip>
  
# Execute following script along with https://customerconnect.vmware.com/ user and password from root directory
$ export TO_URL="http://your-wavefront-url"
$ export TO_TOKEN="your-wavefront-token"
$ export TMC_API_TOKEN="your-tmc-token"
$ ./tkg-install/finish-install.sh <myvmwuser> <myvmwpass>
```
If you set `export SKIP_TSM=true`, the installation will skip installing TSM. If you do not set `TO_TOKEN`, the installation will skip installing Tanzu Observability. If you do not set `TMC_API_TOKEN`, the installation will skip TMC, TO, and TSM.

## Delete Tanzu clusters and aws resources 

```bash
# Login to jumpbox machine and execute command from root directory to delete management and workload clusters
$ ./tkg-install/clean-up.sh

# Execute following command from bootstrap machine to delete all aws resources (vpc , jumpbox etc)
$ terraform destroy
```