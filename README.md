### CloudFormation Template for EC2 with Nginx

This document provides a short explanation of the CloudFormation template created to deploy an EC2 instance running an Nginx web server with the server name **www.signal.co.uk**. It also explains how the template was executed.

### Key Components

- **Parameters:**
  - **KeyName:** Specifies the SSH key pair for accessing the EC2 instance.
  - **InstanceType:** Sets the EC2 instance type (default is `t2.micro`).
  - **LatestAmiId:** Dynamically retrieves the latest Amazon Linux 2 AMI via SSM.

- **Resources:**
  - **EC2 Instance:**  
    Deploys the EC2 instance using the specified AMI and instance type.
    - **UserData:** Runs a bootstrap script that invokes `cfn-init` and `cfn-signal` to execute the configuration set:
      - **Installation:** Enables the Nginx repository and installs Nginx.
      - **Configuration:** Creates a custom Nginx configuration file that sets the server name to **www.signal.co.uk** and generates a simple `index.html` page.
      - **Enablement:** Removes the default Nginx configuration, enables Nginx to start at boot, and restarts the service.
  - **IAM Role and Instance Profile:**  
    Grants the EC2 instance access to CloudFormation metadata during initialization.
  - **Security Group:**  
    Restricts inbound traffic to HTTP (port 80) and SSH (port 22) from a specified IP address (update with your desired IP range).

- **Outputs:**
  - **PublicIp:** Returns the public IP of the deployed instance, which can be used to access the Nginx server.

---

### CloudFormation Execution Role (CFNRole)

The CloudFormation stack assumes this role to create the necessary resources.

- **Trust Policy:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudformation.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

```
- **Permissions Policy**
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"cloudformation:CreateStack",
				"cloudformation:UpdateStack",
				"cloudformation:DeleteStack",
				"cloudformation:DescribeStacks",
				"cloudformation:DescribeStackEvents",
				"cloudformation:GetTemplate",
				"cloudformation:ValidateTemplate"
			],
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": [
				"ec2:RunInstances",
				"ec2:DescribeInstances",
				"ec2:TerminateInstances",
				"ec2:CreateSecurityGroup",
				"ec2:DeleteSecurityGroup",
				"ec2:DescribeSecurityGroups",
				"ec2:AuthorizeSecurityGroupIngress",
				"ec2:AuthorizeSecurityGroupEgress",
				"ec2:DescribeKeyPairs",
				"ec2:DescribeImages",
				"ec2:CreateTags"
			],
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": [
				"iam:CreateRole",
				"iam:DeleteRole",
				"iam:AttachRolePolicy",
				"iam:DetachRolePolicy",
				"iam:PutRolePolicy",
				"iam:DeleteRolePolicy",
				"iam:CreateInstanceProfile",
				"iam:DeleteInstanceProfile",
				"iam:AddRoleToInstanceProfile",
				"iam:RemoveRoleFromInstanceProfile",
				"iam:PassRole"
			],
			"Resource": [
                "arn:aws:iam::<your_account_id>:role/nginx-*",
                "arn:aws:iam::<your_account_id>:role/CFNRole",
                "arn:aws:iam::<your_account_id>:instance-profile/nginx-*"
            ]

		},
		{
			"Effect": "Allow",
			"Action": [
				"ssm:GetParameter",
				"ssm:GetParameters"
			],
			"Resource": "arn:aws:ssm:eu-west-1::parameter/aws/service/ami-amazon-linux-latest/*"
		}
	]
}
```
*Note: Replace <your_account_id> with your actual AWS account ID.*

### Running the CloudFormation Stack

For testing purposes I used an IAM user with the necessary permissions and deployed the stack using an IAM role via the following command:

- **User permissions**

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"cloudformation:CreateStack",
				"cloudformation:UpdateStack",
				"cloudformation:DeleteStack",
				"cloudformation:DescribeStacks",
				"cloudformation:GetTemplate",
				"cloudformation:ValidateTemplate"
			],
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": "sts:AssumeRole",
			"Resource": "arn:aws:iam::<your_account_id>:role/CFNRole"
		},
		{
			"Effect": "Allow",
			"Action": "iam:PassRole",
			"Resource": "arn:aws:iam::<your_account_id>:role/CFNRole"
		}
	]
}
```
*Note: Replace <your_account_id> with your actual AWS account ID.*

- **CLI command**

```sh
aws cloudformation create-stack --stack-name nginx --template-body file://template.yaml --parameters ParameterKey=KeyName,ParameterValue=<yourkeyname> --capabilities CAPABILITY_NAMED_IAM --role-arn "arn:aws:iam::<your_account_id>:role/CFNRole"
```
*Note: Replace vaules with your actual AWS details.*
