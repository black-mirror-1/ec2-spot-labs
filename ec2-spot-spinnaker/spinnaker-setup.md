# Set up a Spinnaker

In this tutorial, we will setup Spinnaker on an EC2 instance, a deployment VPC and setup Spinnaker to deploy to EC2 in that VPC.

## Plannig the Deployment


### AWS Account

If you don’t already have an AWS account, create one at [aws.amazon.com](https://aws.amazon.com) by following the on-screen instructions. Part of the sign-up process involves receiving a phone call and entering a PIN using the phone keypad. Your AWS account is automatically signed up for all AWS services. You are charged only for the services you use.

Before you launch this tutorial, your account must be configured as specified below. Otherwise, the deployment might fail.

### Key pair

Make sure that at least one Amazon EC2 key pair exists in your AWS account in the region where you are planning to deploy the tutorial. Make note of the key pair name. You’ll be prompted for this information during deployment. To create a key pair, follow the [instructions in the AWS documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).
If you’re deploying the tutorial for testing or proof-of-concept purposes, we recommend that you create a new key pair instead of specifying a key pair that’s already being used by a production instance.

### IAM permissions
To deploy the environment, you must log in to the AWS Management Console with IAM permissions for the resources and actions the templates will deploy. You'll need to log in with an IAM User that has the AdministratorAccess managed policy within IAM. This managed policy provides sufficient permissions to deploy this workshop although your organization may choose to use a custom policy with more restrictions.

## Deploy the cloudformation 

We will use the cloudformation template [setup-spinnaker-with-deployment-vpc.yml](templates/setup-spinnaker-with-deployment-vpc.yml) to deploy the below resources.

- A deployment VPC that Spinnaker uses to deploy applications
    - 3 Private Subnets
    - 3 Public Subnets with
- Spinnaker Managed Role as described in [Spinnaker IAM Permissions](https://spinnaker.io/setup/install/providers/aws/#aws-iam-permissions-with-the-aws-cloud-provider)
- IAM role BaseIAMRole which is the default role Spinnaker will use for deployed instances
- Spinnaker installed on an m5.4xlarge EC2 Instance
    - IAM role created and attached with permissions to assume the Spinnaker Managed Role
    - Halyard, a command line administation tools to manage Spinnaker
    - Configure and enable the AWS account to deploy on EC2 using the Spinnaker Managed role using Halyard
    - Choose Persistent Storage for Spinnaker as S3 Using Halyard
    - Enable Launch Template and unlimited CPU Credit specification features as described here (https://spinnaker.io/setup/features/aws-launch-templates/)
- Security groups for demo web application
    - A Security Group for ALB
    - A Security Group for the Server Group (EC2 Instances)

### Open CLoudShell

We will perform the first few of the commands in AWS CloudShell. To lauch the CloudShell click [here](https://console.aws.amazon.com/cloudshell/home) 

### Clone the git repo

```
git clone https://github.com/black-mirror-1/ec2-spot-labs.git
cd ec2-spot-labs/ec2-spot-spinnaker
```

### Deploy the Cloudformation stack
```
STACK_NAME=Spinnaker
SPINNAKER_VERSION=1.25.0
NumberOfAZs=3
REGION=${AWS_DEFAULT_REGION}
AvailabilityZones=${REGION}a,${REGION}b,${REGION}c
EC2KeyPairName=<key name>
```


```
aws cloudformation deploy --template-file templates/setup-spinnaker-with-deployment-vpc.yml --stack-name ${STACK_NAME} --parameter-overrides NumberOfAZs=${NumberOfAZs} AvailabilityZones=${AvailabilityZones}  EC2KeyPairName=${EC2KeyPairName} SpinnakerVersion=${SPINNAKER_VERSION} --capabilities CAPABILITY_NAMED_IAM --region ${REGION}
```

### 

## Connect to Spinnaker

Get your instance's DNS name and [login via SSH](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-2-connect-to-instance.html):

Find the Instance's DNS name from the outputs of the CloudFormation Stack from the AWS console or run the below command in the cloudshell.
```
aws cloudformation describe-stacks --region us-west-2 --stack-name SpinnakerDevInstance --query 'Stacks[0].Outputs[0].OutputValue' --output text
```

Use a terminal on you **local machine** to ssh into the Spinnaker Instance using the below command.

```
ssh -A -L 9000:localhost:9000 -L 8084:localhost:8084 -L 8087:localhost:8087 ubuntu@$spinnaker_instance -i /path/to/my-key-pair.pem
```

