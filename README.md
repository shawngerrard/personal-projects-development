# personal-projects-development
A place to store code relating to personal projects. This is code in development and may get absorbed elsewhere later.

## Prerequisites / Considerations
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

## Install AWS CLI Alias Commands

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

## Create an IAM Administrator User and Group

We need to create an administrator user in AWS IAM to do all our heavy lifting, rather than relying on the root account.

First, lets create the **Administrators** group.

```
aws iam create-group --group-name Administrators
```

Next, lets attach the **AdministratorAccess** permissions policy to the Administrators group.

```
aws iam attach-group-policy --group-name Adminstrators --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```
**Note:** You can confirm the contents of a particular policy with ``aws iam get-policy``

Next, lets create an IAM account to delegate our platform administrative tasks.

```
aws iam create-user --user-name administrator --tags Key="Department",Value="Cloud Platform Services"
```

In this AWS call, I've also added a *Department* tag to the user.

Now, lets give the user access to the AWS Management Console by creating a **login profile** for them.


