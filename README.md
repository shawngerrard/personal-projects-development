# personal-projects-development
A place to store code relating to personal projects. This is code in development and may get absorbed elsewhere later.

The current project is to use AWS CLI to interact with services to spin-up an accessible web server.

<hr>

## Contents
- [Pre-requisites / Considerations](#prereqs)
    - [Install / Update Packages](#packages)
    - [Install AWS CLI Alias Commands](#installcli)
    - [Create an IAM Administrator User and Group](#createadmin)
- [Step 1 - Create and Start an Amazon EC2 Instance](#step1)
    - [Create Amazon EC2 Authentication Key Pairs](#ec2keys)
    - [Create an Amazon EC2 Security Group](#ec2sg)
    - [Create an Inbound Rule for the EC2 Security Group](#sgrule1)
    - [Launch an EC2 Linux Instance](#launchlinuxec2)
    - [Add a Tag to an EC2 Instance](#ec2tag)
- [Step 2 - Connect To and Terminate an Amazon EC2 Instance](#step2)
    - [Connect to an Amazon EC2 Instance](#connec2)
    - [Create a New User Account](#newuserec2)
    - [Terminate an Amazon EC2 Instance](#termec2)
- [(Optional) Step 3 - Start an EC2 Instance using a CloudFormation Template](#step3)

<hr>

## Prerequisites / Considerations <a name='prereqs'></a>

### Install / Update Packages <a name='packages'></a>

1. Ensure your installed modules are up-to-date.

```
sudo apt-get update && sudo apt-get upgrade
```

2. The code in the .dot extension files require use of [GraphvizOnline](https://dreampuf.github.io/GraphvizOnline/) to compile, and is useful for outputting into a variety of visual or code formats to view architectural models.

3. You will need the Python package manager (pip3 for Python v3 or higher) to install the AWS CLI.

```
sudo apt-get install -y python3-pip
```

**Note:** If you follow [James Blair's WSL tooling setup](https://github.com/jmhbnz/tooling/blob/master/wsl-setup.org), this should be already installed.

4. You will need to install the latest version of the [AWS CLI](https://github.com/aws/aws-cli).

```
sudo apt-get install -y awscli
```


### Install AWS CLI Alias Commands <a name='installcli'></a>

I've begun documenting AWS commands I use frequently and have thus created aliases commands for.

Alias commands increase productivity by simplifying frequently used command or Bash script calls.

Feel free to [review my alias library](.aws/cli/README.md) and use the code snippet below to install locally:

```
# Clone/copy the required files from the remote git repository to the current local directory
git clone git@github.com:shawngerrard/personal-projects-development.git

# Create the required directory to place AWS CLI aliases
mkdir -p ~/.aws/cli

# Copy the alias file from the cloned repository to the AWS CLI directory, prompt to overwrite
cp -i personal-projects-development/.aws/cli/alias ~/.aws/cli/alias

# Clean up the redundant repository clone
rm -rf personal-projects-development
```
**Note:** Make sure you backup any existing CLI alias configurations (```mv ~/.aws/cli/alias ~/.aws/cli/alias_backup```) you want to keep before installing this. 


### Create an IAM Administrator User and Group <a name='createadmin'></a>

We need to create an administrator user in AWS IAM to do all our heavy lifting, rather than relying on the root account.

First, lets create the **Administrators** group.

```
aws iam create-group --group-name Administrators
```

Next, lets attach the **AdministratorAccess** permissions policy to the Administrators group.

```
aws iam attach-group-policy --group-name Adminstrators --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```
**Note:** You can confirm the contents of a particular policy with the ``aws iam get-policy`` command.
**Note:** In the future, I may add a section here about [AWS Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) and how to [create them](https://docs.aws.amazon.com/cli/latest/reference/iam/create-role.html).

Next, lets create an IAM account to delegate our platform administrative tasks.

```
aws iam create-user --user-name administrator --tags Key="Department",Value="Cloud Platform Services"
```

In this AWS call, I've also added a *Department* tag to the user.

Next, lets add the new user to the new *Administrator* group we created earlier.

```
aws iam add-user-to-group --group-name "Administrators" --user-name "administrator"
```

Now, lets give the user access to the AWS Management Console by creating a **login profile** for them.

The following code will apply the login profile to the *administrator* profile we created earlier, with the password *#Apple123*.

```
aws iam create-login-profile --user-name administrator --password "#Apple123" --password-reset-required
```

Next, we will also want to provide programmatic access through AWS CLI and API. We do this by creating an access key and attaching it to the user.

```
aws iam create-access-key --user-name administrator
```
**Note:** You may want to store the output (specifically, the access and secret keys) somewhere safe as you won't be able to recover the secret key again.

To be able to use this new user profile in the CLI, we must first add the profile and its access keys to our credentials file (```~/.aws/credentials```).

We can do this programatically with the following code, but first be sure to replace the secret access key placeholder with the secret key provided from the last step.

```
# Add the access key ID to a new administrator profile 
aws configure set aws_access_key_id `aws iam list-access-keys --user-name administrator --query 'AccessKeyMetadata[*].AccessKeyId' --output text` --profile administrator

# Add the secret access key from the previous step to the administrator profile
aws configure set aws_secret_access_key <Enter Secret Key Here> --profile administrator
```

Now, you should be able to use this profile to make AWS resource calls, E.G: ```aws s3 ls --profile administrator```.

I recommend removing access keys for the root user and replacing your default profile (in your configuration and credentials files under ```~/.aws```) with the *administrator* profile we've just created.

**Note:** To use this account on another machine, you will need to run the code block above to set the AWS access key ID and secret key that you generated in the previous steps.

<hr>

## Step 1 - Create and Start an Amazon EC2 Instance <a name="ec2instance"></a>


### Create Authentication Key Pairs <a name="ec2keys"></a>

To connect and interact with Amazon EC2 instances over SSH, we must first create SSH keys that will authenticate specific Amazon/AWS resources (in this case, EC2) with the current local machine.

You could use the ```ssh-keygen``` command to generate the key pairs, then import them into AWS using ```aws ec2 import-key-pair --key-name "my-key" --public-key-material fileb://~/.ssh/my-key.pub```. However, we're going to use AWS to generate the keys and import them. We do this with the following command:

```
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > ~/.ssh/MyKeyPair.pem
```

This command will create a 2048-bit RSA key pair, import the public key into Amazon EC2, and pipe the private key output into a private key file that is saved in your *.ssh* folder.

We will also set the permissions on the key file to ensure that it can only be read by this current user.

```
chmod 400 MyKeyPair.pem
```

To display the SHA1 hash fingerprint of the public key stored in AWS, you can use the following command:

```
aws ec2 describe-key-pairs --key-name MyKeyPair --query 'KeyPairs[*].[KeyName,KeyFingerprint]' --output text
```
**Note:** You can remove the ```--key-name``` option to list all keys, or use ```grep``` and/or ```awk``` on the key name.


### Create an Amazon EC2 Security Group <a name="ec2sg1"></a>

Now that we have a way to authenticate with an Amazon EC2 instance, we need to configure a [Security Group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) to control the flow of inbound/outbound traffic to/from an EC2 instance.

**Note:** By default, Security Groups deny all inbound and allow all outbound traffic.

To create a Security Group, first think of a good name for it (I've used the template *<username/role>_sg_<region>* to create "administrator_sg_apsoutheast2"), and then run the following code to create it with this name and also a brief description.

```
aws ec2 create-security-group --group-name administrator_sg_apsoutheast2 --description "Security Group for Amazon EC2 instance 1" --tag-specifications ResourceType="security-group"
```

### Add an Inbound Rule to the EC2 Security Group <a name="sgrule1"></a>

Next, we need to create and add rules to the Security Group.

The first rule we'll define will allow *SSH traffic on port 22* originating from your IP address to enter. Use the following code to create the rule and attach it to the security group we've created.

```
aws ec2 authorize-security-group-ingress --group-name administrator_sg_apsoutheast2 --protocol tcp --port 22 --cidr `curl https://checkip.amazonaws.com | awk '{ sub(/[.]([0-9]{2,3})$/,"&/32"); print }'`
```
**Note:** The end of this command requests your external IP address from Amazon and creates a [Class C CIDR address mask](https://www.watchguard.com/wgrd-resource-center/security-fundamentals/understanding-ipv4-subnetting-part-one) from it.
For example - if Amazon returned my external IP address as *121.99.164.101*, the command will transform that address into *121.99.164.101/32* and set the ```--cidr``` attribute with this value.

**Note:** The ```/32``` part of the CIDR IP address notation indicates to AWS to use your individual IP address. You could set a range of IP's by changing the CIDR number, however you must be careful doing this on external IP's due to the ambiguity of the netmask that the ISP uses to allocate your IP address.

**Note:** In the future, I may add instructions here to create a template function that will update your external IP, as well as a cronjob or alternative method that will periodically initiate the function.

Now, we will allow all inbound traffic over HTTP and HTTPS with the following code:

```
# Allow all inbound traffic over HTTPS
aws ec2 authorize-security-group-ingress --group-name administrator_sg_apsoutheast2 --protocol tcp --port 443 --cidr 0.0.0.0/0

# Allow all inbound traffic over HTTP
aws ec2 authorize-security-group-ingress --group-name administrator_sg_apsoutheast2 --protocol tcp --port 80 --cidr 0.0.0.0/0
```


### Launch an EC2 Linux Instance <a name="launchlinuxec2"></a>

At last - we're ready to configure and launch an AWS EC2 Linux instance!

To launch an instance, we **must** specify the following details with the launch call:
- The Amazon Machine Image (AMI) to launch the instance with.
- The instance type we want to launch.
- The private key (that we crreated earlier) to use when connecting to the instannce via SSH. 

The AMI is a template that contains software configurations. These configurations set what operating system, application servers, and/or applications we want to install into the instance.

Amazon has a large library of pre-configured AMI's, however users may create a customized AMI according to needs.

For the purposes of this example, we'll use a preconfigured, public AMI from Amazon that will be suitable for both our use-case and also our AWS free-tier constraints.

You can use ```aws ec2 describe-images --owners self amazon``` with filter options to display particular AMI's, or access the AMI section in the [Amazon EC2 console](https://console.aws.amazon.com/ec2/).

For this example, the AMI we'll use has the following attributes:

| Attribute | Value |
| ----------| ----- |
| AMI ID | ami-0210560cedcb09f07 |
| AMI Name | Amazon Linux 2 AMI |
| Platform | Amazon Linux |
| CPU Architecture | x86-64 |
| Virtualisation | HVM |
| Root Device Type | Elastic Block Storage (EBS) |

We can launch this EC2 instance with the following code:

```
aws ec2 run-instances --image-id resolve:ssm:/aws/service/ami-amazon-linux-latest/amzn-ami-hvm-x86_64-gp2 --count 1 --instance-type t2.micro --key-name <Key Name as per above> --security-group-ids `aws get-security-group-id administrator_sg_apsoutheast2`
```

This code will do the following:
- Use the [Systems Manager (SSM) public parameter](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html#finding-an-ami-parameter-store) to retrieve the latest Amazon Linux 2 x86-64 image.
**Note:** You can update this parameter to specifically target a version if you want to control version migration.
- Create a single instance.
- Define the instance type as [T2.Micro](https://aws.amazon.com/ec2/instance-types/).
- Define the SSH key-pair to use to connect to the instance - in this case, we should be using the SSH key created earlier.
- Retrieve the ID of the security group we created earlier, and attach the security group to the instance.
- Launch the instance.

If you're using the [Alias file](.aws/cli/alias) I've included in this repository, you'll be able to use the ```aws describe-ec2-states``` to check for when the instance is running.


### Add a Tag to an EC2 Instance <a name="ec2tag"></a>

Next, we want to add a tag that can easily identify the new EC2 instance, as instances do not have a name field by default.

```
aws ec2 create-tags --resources `aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro" --query "Reservations[].Instances[].InstanceId" --output text` --tags Key=Name,Value=awsec2-administrator
```

This will set the tag **Name** with the value **awsec2-administrator**. We can reference this instance now with the following code:

```
aws ec2 describe-instances --filters "Name=tag:Name,Values=awsec2-administrator"
```

Using the Alias function set in my [Alias file](.aws/cli/alias), you can retrieve the instance we've just created by the tag we've just created by using the following code:

```
aws get-instance-by-tag Name awsec2-administrator
```

<hr>

## Step 2 - Connect and Terminate an Amazon EC2 Instance<a name="conntermec2"></a>

### Connect to an Amazon EC2 Instance <a name="connec2"></a>

Now that the instance is launched, we can connect to it and use it the way you'd use a computer.

Because we've allowed SSH traffic into the instance, we can connect to it using SSH.

```
ssh -i <path/to/key>.pem ec2-user@`aws ec2 describe-instances --filters "Name=tag:Name,Values=awsec2-administrator" --query 'Reservations[].Instances[].PublicDnsName' --output text`
```
**Note:** Replace the path to the key with the key path that you create earlier.


### Create a New User Account in EC2 <a name="newuserec2"></a>

Before we create a new user, we need to create a new key-pair to allow this user to connect to this instance.

You can create this key-pair by [using the instructions we followed earlier](#ec2keys). You can create the key-pair anywhere you like, but do be aware you will need to use the contents of the file in the next few steps.

Once you've generated a new key-pair, retrieve the contents of the public key by running the following code on the computer holding it.

```
ssh-keygen -y -f ~/.ssh/admin_ec2_user.pem | xclip
```

In the instance terminal, let's now create a new user account.

```
sudo adduser administrator
```

**Note:** This will create a group and home directory for the account.

Now, let's switch to the new account.

```
sudo su - administrator
```

We need to create an ```.ssh``` directory for the new administrator account and change its permissions so that only the owner can read/write and open the directory.

```
# Create the .ssh directory
mkdir ~/.ssh

# Change the permissions on the directory
chmod 700 ~/.ssh
```

Next, we want to create a file named ```authorized_keys``` and change its file permissions so that **only the owner can read/write to the file**

```
# Create an empty file in .ssh
touch ~/.ssh/authorized_keys

# Update permissions on the file
chmod 600 ~/.ssh/authorized_keys
```

Now we need to update the new *authorized_keys* file with the public key data we copied earlier.

```

```



### Terminate an Amazon EC2 Instance <a name="termec2"></a>

Use the following code to terminate the Amazon EC2 instance we've started.

```
aws ec2 terminate-instances --instance-ids --filter "Name=tag:Name,Values=awsec2-administrator" --query 'Reservations[].Instances[].InstanceId'
```

<hr>

## (Optional) Step 3 - Start an EC2 Instance Using a CloudFormation Template <a name="step3"></a>

While this section is optional, it's recommended to follow this step for the following reasons:
1. CloudFormation templates manage your infrastructure as code and allow you to automate deployments.
2. CloudFormation templates, with the use of [User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-add-user-data), rudimentally addresses the problems relating to secured key-based authentication of SSH access for multiple users to a single EC2 instance - you cannot do this by default using purely AWS CLI.
3. CloudFormation templates provide a single source of truth for infrastructure deployments.



