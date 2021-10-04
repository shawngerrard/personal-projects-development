# AWS CLI Configuration and Alias File

You can create an alias file under **~/.aws/cli** (if one doesn't already exist) that can host aliases to be used to call functions or scripts easier, improving CLI productivity.

I've included my own alias file to share the functions I've found useful while navigating AWS CLI.

Below are the aliases I've used in order of precedence. You must use the 'aws' prefix before the alias to use it.

```aws [alias]``` E.G ```aws whoami```

- [delete-user](#delete-user) <username>
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

## <a name="get-region"></a>get-region

This function returns the region currently set for this AWS CLI session.

<hr>

## <a name="get-security-group-id"></a>get-security-group-id

This function returns the security group ID from a specified security group name.

Use the following code to use the function correctly:

```
aws ec2 describe-security-groups --group-name <security group name> --query 'SecurityGroups[].GroupId' --output text
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
