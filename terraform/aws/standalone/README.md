# Check Point CloudGuard Network Security Management Server & Security Gateway (Standalone) Terraform module for AWS

Terraform module which deploys a Check Point CloudGuard Network Security Gateway & Management (Standalone) instance into an existing VPC.

These types of Terraform resources are supported:
* [AWS Instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)
* [Security group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)
* [Network interface](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/network_interface)
* [EIP](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/eip) - conditional creation
* [Route](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route) - conditional creation


This solution uses the following modules:
- /terraform/aws/modules/amis

## Configurations

The **main.tf** file includes the following provider configuration block used to configure the credentials for the authentication with AWS, as well as a default region for your resources:
```
provider "aws" {
  region = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}
```
The provider credentials can be provided either as static credentials or as [Environment Variables](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#environment-variables).
- Static credentials can be provided by adding an access_key and secret_key in /terraform/aws/standalone/**terraform.tfvars** file as follows:
```
region     = "us-east-1"
access_key = "my-access-key"
secret_key = "my-secret-key"
```
- In case the Environment Variables are used, perform modifications described below:<br/>
  a. The next lines in main.tf file, in the provider aws resource, need to be commented:
  ```
  provider "aws" {
  //  region = var.region
  //  access_key = var.access_key
  //  secret_key = var.secret_key
  }
  ```

## Usage
- Fill all variables in the /terraform/aws/standalone/**terraform.tfvars** file with proper values (see below for variables descriptions).
- From a command line initialize the Terraform configuration directory:
    ```
    terraform init
    ```
- Create an execution plan:
    ```
    terraform plan
    ```
- Create or modify the deployment:
    ```
    terraform apply
    ```
  
- Variables are configured in /terraform/aws/standalone/**terraform.tfvars** file as follows:

  ```
    //PLEASE refer to README.md for accepted values FOR THE VARIABLES BELOW

    // --- VPC Network Configuration ---
    vpc_id = "vpc-12345678"
    public_subnet_id = "subnet-123456"
    private_subnet_id = "subnet-345678"
    private_route_table = "rtb-12345678"

    // --- EC2 Instance Configuration ---
    standalone_name = "Check-Point-Standalone-tf"
    standalone_instance_type = "c5.xlarge"
    key_name = "privatekey"
    allocate_and_associate_eip = true
    volume_size = 100
    volume_encryption = "alias/aws/ebs"
    enable_instance_connect = false
    instance_tags = {
      key1 = "value1"
      key2 = "value2"
    }

    // --- Check Point Settings ---
    standalone_version = "R81-BYOL"
    admin_shell = "/bin/bash"
    standalone_password_hash = "12345678"

    // --- Advanced Settings ---
    resources_tag_name = "tag-name"
    standalone_hostname = "standalone-tf"
    allow_upload_download = true
    standalone_bootstrap_script = "echo 'this is bootstrap script' > /home/admin/testfile.txt"
    primary_ntp = ""
    secondary_ntp = ""
    admin_cidr = "0.0.0.0/0"
    gateway_addresses = "0.0.0.0/0"
  ```

- Conditional creation
  - To create an Elastic IP and associate it to the Standalone instance:
  ```
  allocate_and_associate_eip = true
  ```
  - To create route from '0.0.0.0/0' to the Standalone instance, please provide route table:
  ```
  private_route_table = "rtb-12345678"
  ```
- To tear down your resources:
    ```
    terraform destroy
    ```


## Inputs
| Name          | Description   | Type          | Allowed values | Default       | Required      |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| vpc_id | The VPC id in which to deploy | string | n/a | n/a | yes |
| public_subnet_id | The public subnet of the Security Gateway & Management (Standalone) | string | n/a | n/a | yes |
| private_subnet_id | The private subnet of the Security Gateway & Management (Standalone) | string | n/a | n/a | yes |
| private_route_table | Sets '0.0.0.0/0' route to the Security Gateway & Management (Standalone) instance in the specified route table (e.g. rtb-12a34567) | string | n/a | "" | no |
| standalone_name | (Optional) The name tag of the Security Gateway & Management (Standalone) instance | string | n/a | Check-Point-Standalone-tf | no |
| standalone_instance_type | The instance type of the Security Gateway & Management (Standalone) instance | string | - c5.large <br/> - c5.xlarge <br/> - c5.2xlarge <br/> - c5.4xlarge <br/> - c5.9xlarge <br/> - c5.18xlarge <br/> - c5n.large <br/> - c5n.xlarge <br/> - c5n.2xlarge <br/> - c5n.4xlarge <br/> - c5n.9xlarge <br/> - c5n.18xlarge <br/> - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.12xlarge <br/> - m5.24xlarge| c5.xlarge | no |
| key_name | The EC2 Key Pair name to allow SSH access to the instances | string  | n/a | n/a | yes |
| allocate_and_associate_eip | If set to true, an elastic IP will be allocated and associated with the launched instance | bool | true/false | true | no |
| volume_size | Root volume size (GB) - minimum 100 | number | n/a | 100 | no |
| volume_encryption | KMS or CMK key Identifier: Use key ID, alias or ARN. Key alias should be prefixed with 'alias/' (e.g. for KMS default alias 'aws/ebs' - insert 'alias/aws/ebs') | string | n/a | alias/aws/ebs | no |
| enable_instance_connect | Enable SSH connection over AWS web console. Supporting regions can be found [here](https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/) | bool | true/false | false | no |
| instance_tags  | (Optional) A map of tags as key=value pairs. All tags will be added to the Standalone EC2 Instance | map(string)  | n/a  | {}  | no  |
| standalone_version | Security Gateway & Management (Standalone) version and license | string | - R80.40-PAYG-NGTP <br/> - R81-PAYG-NGTP <br/> - R81.10-PAYG-NGTP | R81-BYOL | no |
| admin_shell | Set the admin shell to enable advanced command line configuration | string | - /etc/cli.sh <br/> - /bin/bash <br/> - /bin/csh <br/> - /bin/tcsh | /etc/cli.sh | no |
| standalone_password_hash | (Optional) Admin user's password hash (use command 'openssl passwd -6 PASSWORD' to get the PASSWORD's hash) | string | n/a | "" | no |
| resources_tag_name | (optional) | string | n/a | "" | no |
| standalone_hostname | (Optional) Security Gateway & Management (Standalone) prompt hostname | string | n/a | "" | no |
| allow_upload_download | Automatically download Blade Contracts and other important data. Improve product experience by sending data to Check Point | bool | true/false | true | no |
| standalone_bootstrap_script | (Optional) Semicolon (;) separated commands to run on the initial boot | string | n/a | "" | no |
| primary_ntp | (Optional) The IPv4 addresses of Network Time Protocol primary server | string | n/a | 169.254.169.123 | no |
| secondary_ntp | (Optional) The IPv4 addresses of Network Time Protocol secondary server | string | n/a | 0.pool.ntp.org | no |
| admin_cidr | (CIDR) Allow web, ssh, and graphical clients only from this network to communicate with the Management Server | string | n/a | 0.0.0.0/0 | no |
| gateway_addresses | (CIDR) Allow gateways only from this network to communicate with the Management Server | string | n/a | 0.0.0.0/0 | no |


## Outputs
| Name  | Description |
| ------------- | ------------- |
| standalone_instance_id  | The deployed Security Gateway & Management (Standalone) AWS instance id |
| standalone_instance_name  | The deployed Security Gateway & Management (Standalone) AWS instance name  |
| standalone_public_ip  | The deployed Security Gateway & Management (Standalone) AWS public address  |
| standalone_ssh  | SSH command to the Security Gateway & Management (Standalone) |
| standalone_url  | URL to the portal of the deployed Security Gateway & Management (Standalone)  |

## Revision History
In order to check the template version, please refer to [sk116585](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk116585)

| Template Version | Description   |
| ---------------- | ------------- |
| 20210309 | First release of Check Point Security Management Server & Security Gateway (Standalone) Terraform module for AWS |
| 20210329 | Stability fixes |



## License

This project is licensed under the MIT License - see the [LICENSE](../../LICENSE) file for details