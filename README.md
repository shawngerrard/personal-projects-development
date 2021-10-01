# personal-projects-development
A place to store code relating to personal projects. This is code in development and may get absorbed elsewhere later.

The current project is to use AWS CLI to interact with services to spin-up an accessible web server.


## Contents
- [Pre-requisites / Considerations](#prereqs)
    - [Install / Update Packages](#packages)
    - [Install AWS CLI Alias Commands](#installcli)
    - [Create an IAM Administrator User and Group](#createadmin)
- [Step 1 - Create and Start an Amazon EC2 Instance](#ec2instance)
    - [Create Amazon EC2 Authentication Key Pairs](#ec2keys)
    - [Create an Amazon EC2 Security Group](#ec2sg)

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


### Create an Amazon EC2 Security Group <a name="ec2sg"></a>

Now that we have a way to authenticate with an Amazon EC2 instance, we need to configure a [Security Group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) to control the flow of inbound/outbound traffic to/from an EC2 instance.

**Note:** By default, Security Groups deny all inbound and allow all outbound traffic.

To create a Security Group, first think of a good name for it (I've used the template *<username/role>_sg_<region>* to create "administrator_sg_apsoutheast2"), and then run the following code to create it with this name and also a brief description.

```
aws ec2 create-security-group --group-name administrator_sg_apsoutheast2 --description "Security Group for Amazon EC2 instance 1"
```


