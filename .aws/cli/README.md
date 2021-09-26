# AWS CLI Configuration and Alias File

You can create an alias file under **~/.aws/cli** (if one doesn't already exist) that can host aliases to be used to call functions or scripts easier, improving CLI productivity.

I've included my own alias file to share the functions I've found useful while navigating AWS CLI.

Below are the aliases I've used in order of precedence. You must use the 'aws' prefix before the alias to use it.

```aws [alias]``` E.G ```aws whoami```

- [get-region](#get-region)
- [whoami](#whoami)

## <a name="get-region"></a>get-region

This function returns the region currently set for this AWS CLI session.

## <a name="whoami"></a>whoami

This function returns the user account currently active in the AWS CLI session.
