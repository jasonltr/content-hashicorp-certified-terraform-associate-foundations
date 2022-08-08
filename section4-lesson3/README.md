#This folder is for the Terraform Workspace lesson code.

# The main.tf file contains provider and EC2 VM resources
# and uses ${terraform.workspace} variable and logic to decide what 
# region to deploy in

#The network.tf file spins up the network resources required by
#the EC2 VM and uses ${terraform.workspace} variable to set their Names/IDs.


## my attempt

terraform lab in ACG  
[section4-lesson3](/section4-lesson3/)  
spin up aws sandbox instance  
run `aws configure` to set up credentials  
download [tfenv](https://github.com/tfutils/tfenv) to manage different terraform versions  
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ aws configure
AWS Access Key ID [****************OICA]: AKIATH4OMHHYQCM3IIR3
AWS Secret Access Key [****************b402]: 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
Default region name [us-east-1]: us-east-1
Default output format [json]: 
```
check terraform version with `tfenv list`, `tfenv use` then `terraform version`  
read through [main.tf](section4-lesson3/main.tf) and [network.tf](section4-lesson3/network.tf) to understand what you will be building  
we will create a separate workspace for this 
Note the configurations in the main.tf code, particularly:  
AWS is the selected provider.  
If the code is deployed on the default workspace, the resources will be deployed in the us-east-1 region.  
If the code is deployed on any other workspace, the resources will be deployed in the us-west-2 region.  
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace list
* default

jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace new test
Created and switched to workspace "test"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace list
  default
* test
```
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v4.25.0...
- Installed hashicorp/aws v4.25.0 (self-signed, key ID 34365D9472D7468F)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform apply --auto-approve
aws_vpc.vpc_master: Creating...
aws_vpc.vpc_master: Creation complete after 3s [id=vpc-080f346da75383cd6]
aws_subnet.subnet: Creating...
aws_security_group.sg: Creating...
aws_subnet.subnet: Creation complete after 2s [id=subnet-05c6780ec3604246c]
aws_security_group.sg: Creation complete after 5s [id=sg-0c0136085de6093db]
aws_instance.ec2-vm: Creating...
aws_instance.ec2-vm: Still creating... [10s elapsed]
aws_instance.ec2-vm: Creation complete after 14s [id=i-0e69cc1bed202be54]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```
![](/section4-lesson3/images/terraform_1.png)
resources are deployed in the us-west-2 region as the workspace is in test/not default  
check terraform state list
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform state list
data.aws_availability_zones.azs
data.aws_ssm_parameter.linuxAmi
aws_instance.ec2-vm
aws_security_group.sg
aws_subnet.subnet
aws_vpc.vpc_master
```
Now we deploy onto default workspace
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace select default
Switched to workspace "default".
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform state list
No state file was found!

State management commands require a state file. Run this command
in a directory where Terraform has been run or use the -state flag
to point the command to a specific state location.
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace list
* default
  test
```
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform apply --auto-approve
aws_vpc.vpc_master: Creating...
aws_vpc.vpc_master: Creation complete after 4s [id=vpc-0d8c6267d518ebbe2]
aws_subnet.subnet: Creating...
aws_security_group.sg: Creating...
aws_subnet.subnet: Creation complete after 2s [id=subnet-09821c2a2f47ed9a4]
aws_security_group.sg: Creation complete after 5s [id=sg-051e16e30890ff6a2]
aws_instance.ec2-vm: Creating...
aws_instance.ec2-vm: Still creating... [10s elapsed]
aws_instance.ec2-vm: Creation complete after 17s [id=i-0cf929d941b73070a]

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.
```
verify this in aws console, change location to us-east-1  
![](/section4-lesson3/images/terraform_2.png)
Now we destory whatever was created  
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace select test
Switched to workspace "test".
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform destroy --auto-approve
aws_instance.ec2-vm: Destroying... [id=i-0e69cc1bed202be54]
aws_instance.ec2-vm: Still destroying... [id=i-0e69cc1bed202be54, 10s elapsed]
aws_instance.ec2-vm: Still destroying... [id=i-0e69cc1bed202be54, 20s elapsed]
aws_instance.ec2-vm: Still destroying... [id=i-0e69cc1bed202be54, 30s elapsed]
aws_instance.ec2-vm: Still destroying... [id=i-0e69cc1bed202be54, 40s elapsed]
aws_instance.ec2-vm: Destruction complete after 42s
aws_subnet.subnet: Destroying... [id=subnet-05c6780ec3604246c]
aws_security_group.sg: Destroying... [id=sg-0c0136085de6093db]
aws_subnet.subnet: Destruction complete after 1s
aws_security_group.sg: Destruction complete after 2s
aws_vpc.vpc_master: Destroying... [id=vpc-080f346da75383cd6]
aws_vpc.vpc_master: Destruction complete after 0s

Destroy complete! Resources: 4 destroyed.
```
check aws console  
![](/section4-lesson3/images/terraform_3.png)
delete the test workspace
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace delete test
Deleted workspace "test"!
```
Delete the resources created in default workspace
```
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform workspace select default
Switched to workspace "default".
jason@DEV-52WP6M3:~/Documents/content-hashicorp-certified-terraform-associate-foundations/section4-lesson3$ terraform destroy --auto-approve

aws_instance.ec2-vm: Destroying... [id=i-0cf929d941b73070a]
aws_instance.ec2-vm: Still destroying... [id=i-0cf929d941b73070a, 10s elapsed]
aws_instance.ec2-vm: Still destroying... [id=i-0cf929d941b73070a, 20s elapsed]
aws_instance.ec2-vm: Still destroying... [id=i-0cf929d941b73070a, 30s elapsed]
aws_instance.ec2-vm: Still destroying... [id=i-0cf929d941b73070a, 40s elapsed]
aws_instance.ec2-vm: Still destroying... [id=i-0cf929d941b73070a, 50s elapsed]
aws_instance.ec2-vm: Destruction complete after 52s
aws_subnet.subnet: Destroying... [id=subnet-09821c2a2f47ed9a4]
aws_security_group.sg: Destroying... [id=sg-051e16e30890ff6a2]
aws_subnet.subnet: Destruction complete after 2s
aws_security_group.sg: Destruction complete after 2s
aws_vpc.vpc_master: Destroying... [id=vpc-0d8c6267d518ebbe2]
aws_vpc.vpc_master: Destruction complete after 1s

Destroy complete! Resources: 4 destroyed.
```
verify on aws console  
![](/section4-lesson3/images/terraform_4.png)
  
lab done