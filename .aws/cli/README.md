# AWS CLI Configuration and Alias File

You can create an alias file under **~/.aws/cli** (if one doesn't already exist) that can host aliases to be used to call functions or scripts easier, improving CLI productivity.

I've included my own alias file to share the functions I've found useful while navigating AWS CLI.

Below are the aliases I've used in order of precedence. You must use the 'aws' prefix before the alias to use it.

```aws [alias]``` E.G ```aws whoami```

- [delete-user](#delete-user) <username>
- [describe-ec2-states](#describe-ec2-states)
- [get-instance-by-tag](#get-instance-by-tag) <tag name> <tag value>
- [get-region](#get-region)
- [get-security-group-id](#get-sg-id) <security group name>
- [revoke-all-ec2-ingress-rules](#revoke-all-ec2-ingress-rules) <security group name>
- [whoami](#whoami)

<hr>

## <a name="delete-user"></a>delete-user

**Rationale:** Before AWS can delete an IAM user, all related dependencies to the user must also be removed/deleted.

This function removes and/or deletes all of the user-related dependencies (groups, policies, login profiles, etc) before finally deleting the IAM user.

```
aws delete-user "shawng"
```

<hr>

## <a name="describe-ec2-states"></a>describe-ec2-states

**Rationale:** There doesn't seem to be an easy method to retrieve the state of all EC2 instances.

This function returns a list of all active instances and their current state (E.G - pending, running, terminated), as well as their public DNS name if one has been allocated by Amazon.

This is useful when you want to track the state status of instances, for example - when you first launch an instance.

<hr>

## <a name="get-instance-by-tag"></a>get-instance-by-tag

**Rationale:** Simplifies the referencing of EC2 instance tags.

This function retrieves the instances that meet the specified tag requirements.

```
#aws get-instance-by-tag <tag name> <tag value>
aws get-instance-by-tag Name AWSEC2-Administrator
```

<hr>

## <a name="get-region"></a>get-region

This function returns the region currently set for this AWS CLI session.

<hr>

## <a name="get-sg-id"></a>get-security-group-id

This function returns the security group ID from a specified security group name.

Use the following code to use the function correctly:

```
aws get-security-group-id <security group name>
```

<hr>

## <a name="revoke-all-ec2-ingress-rules"></a>revoke-all-ec2-ingress-rules

This function will revoke *all* ingress rules on a specified security group name.

Use the following code to use the function correctly:

```
aws revoke-all-ec2-ingress-rules <group name>
```

<hr>

## <a name="whoami"></a>whoami

This function returns the user account currently active in the AWS CLI session.
