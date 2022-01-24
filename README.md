# Manual Account Setup

test

test

test

Before you can use the terraform templates as they're intended you will need to manually configure the accounts' IAM principals for programmatic access.

Ideally you will create one account for each environment, plus one account to serve as your administrative account where you will host the Terraform backend. In this manual we will assume three environments `dev`, `int`, `prd` plus one administrative "environment" that we shall call `tfm`.

## 1. Administrative Account `tfm` - Create the IAM User

1.  Determine which account will serve as your administrative account and take note of the 12-digit account number. Let's pretend this to be `000000000000` in our example. If you're not equipped with any existing account, create them.

    > If you just created the account make sure to create an IAM user with admin permissions and Management Console access so you don't rely on your root user to access the MC.
2. Create an AWS IAM user `tf-user` (or any other name you please) with AWS CLI access from the MC. After successful creation store the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in a safe place like a password manager.
3. After you created the user revisit it and set the necessary permissions in a policy. This policy should cover all actions required to deploy the `remote_state` terraform module in this project and all actions required to read and write the terraform backend. If you're unsure, start by simply attaching an \`AdministratorAccess\` managed policy, but remember: with great power comes great responsibility.
4. Create a local AWS CLI profile for this account:
   1.  In your `~/.aws/config` file add the following entry:

       ```
       [profile tf-user]
       region = eu-central-1
       output = json
       ```
   2.  Look up your `tf-user` credentials again and insert them into your `~/.aws/credentials` file under the profile name we just created, e.g. like so (for ficticious AWS user credentials):

       ```
       [tf-user]
       aws_access_key_id = AAAAAAAAAAAAAAAAAAAA
       aws_secret_access_key = +abcdefghijklmnopqrstuvwxyz1234321ASDFGH
       ```
   3.  Verify your profile works as expected by performing the following commands in your terminal:

       ```
       $ export AWS_PROFILE=tf-user
       $ aws sts get-caller-identity
       {
           "UserId": "AAAAAAAAAAAAAAAAAAAA",
           "Account": "000000000000",
           "Arn": "arn:aws:iam::000000000000:user/tf-user"
       }
       ```

## 2. Environment Accounts `dev`, `int`, `prd` - Create the IAM Roles

1.  Determine how your other accounts should tally with the development environments `dev`, `int`, `prd`. Remember the account numbers. We will pretend to have the following mapping in this example:

    ```
    dev = 111111111111
    int = 222222222222
    prd = 333333333333
    ```

    If you're not equipped with any existing accounts, create them.
2. In each of these accounts perform the following steps:
   1. Login into the MC and create an IAM role `tf-role` (or any other name you please).
   2. After you created the role revisit it and set the sensible permissions in a role policy. This policy should cover all actions required to deploy the AWS resources in your contemplated IAC project. If you're unsure, start by simply attaching an `AdministratorAccess` managed policy, but remember: with great power comes great responsibility.
   3.  Lastly, establish a trust relationship between the `tf-user` you created in the administrative account earlier. You should do this by referencing the `tf-user` as a principal, not the administirative `tfm`-account itself. The trust relationship policy document (for every version of this role in each account) should look like this:

       ```json
       {
           "Version": "2012-10-17",
           "Statement": [
               {
                   "Effect": "Allow",
                   "Principal": {
                       "AWS": "arn:aws:iam::000000000000:user/tf-user"
                   },
                   "Action": "sts:AssumeRole",
                   "Condition": {}
               }
           ]
       }
       ```

## 3. Administrative Account `tfm` - IAM User Consent

Finally, after we have created the trust relationships with the `tfm`-account's `tf-user` we need to implement the `tf-user`'s consent as well.

1. Go back to the MC of your administrative `tfm`-account, and revisit the `tf-user`.
2. test
3.  Attach a new policy with a policy document allowing the `sts:AssumeRole` action for each `tf-role` in their corresponding environment accounts. It should look like this:

    ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": "sts:AssumeRole",
                 "Resource": "arn:aws:iam::111111111111:role/tf-role"
             },
             {
                 "Effect": "Allow",
                 "Action": "sts:AssumeRole",
                 "Resource": "arn:aws:iam::222222222222:role/tf-role"
             },
             {
                 "Effect": "Allow",
                 "Action": "sts:AssumeRole",
                 "Resource": "arn:aws:iam::333333333333:role/tf-role"
             }
         ]
     }
    ```
