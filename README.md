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
git clone git@github.com:shawngerrard/personal-projects-development.git
mkdir -p ~/.aws/cli
cp -i awscli-aliases/alias ~/.aws/cli/alias
rm -rf personal-projects-development
```
**Note:** Make sure you backup any existing CLI alias configurations (```nano ~/.aws/cli/alias```) you want to keep before installing this. 
