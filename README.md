# Gimme AWS Creds

gimme-aws-creds is a CLI that utilizes an [Okta](https://www.okta.com/) IdP via SAML to acquire temporary AWS credentials via AWS STS.

Okta is a SAML identity provider (IdP), that can be easily set-up to do SSO to your AWS console. Okta does offer an [OSS java CLI]((https://github.com/oktadeveloper/okta-aws-cli-assume-role)) tool to obtain temporary AWS credentials, but I found it needs more information than the average Okta user would have and doesn't scale well if have more than one Okta App.

With gimme-aws-creds all you need to know is your username, password, Okta url and MFA token, if MFA is enabled. gimme-aws-creds gives you the option to select which Okta AWS application and role you want credentials for. Alternatively, you can pre-configure the app and role name by passing -c or editing the config file. This is all covered in the usage section.


## Installation
This is a Python 3 project.


Install/Upgrade the latest gimme-aws-creds package direct from GitHub:
```bash
pip3 install --upgrade git+git://github.com/IntegralReach/gimme-aws-creds.git
```


## Configuration

To set-up the configuration run:
```bash
gimme-aws-creds --configure
```

You can also set up different Okta configuration profiles, this useful if you have multiple Okta accounts or environments you need credentials for. You can use the configuration wizard or run:
```bash
gimme-aws-creds --configure --profile profileName
```

A configuration wizard will prompt you to enter the necessary configuration parameters for the tool to run, the only one that is required is the `okta_org_url`. The configuration file is written to `~/.okta_aws_login_config`, but you can change the location with the environment variable `OKTA_CONFIG`.

- conf_profile - This sets the Okta configuration profile name, the default is DEFAULT.
- okta_org_url - This is your Okta organization url, which is typically something like `https://tivo.okta.com`.

- gimme_creds_server - This is optional
	- URL for gimme-creds-lambda
	- 'internal' for direct interaction with the Okta APIs (`OKTA_API_KEY` environment variable required)
	- 'appurl' to set an aws application link url. This setting removes the need of an OKTA API key.
- write_aws_creds - True or False - If True, the AWS credentials will be written to `~/.aws/credentials` otherwise it will be written to stdout.
- cred_profile - If writing to the AWS cred file, this sets the name of the AWS credential profile.  The reserved word 'role' will use the name component of the role arn as the profile name.  i.e. arn:aws:iam::123456789012:role/okta-1234-role becomes section [okta-1234-role] in the aws credentials file
- aws_appname - This is optional. The Okta AWS App name, which has the role you want to assume.
- aws_rolename - This is optional. The ARN of the role you want temporary AWS credentials for.  The reserved word 'all' can be used to get and store credentials for every role the user is permissioned for.
- aws_default_duration = This is optional. Lifetime for temporary credentials, in seconds. Defaults to 1 hour (3600)
- app_url - If using 'appurl' setting for gimme_creds_server, this sets the url to the aws application configured in Okta. It is typically something like https://something.okta[preview].com/home/amazon_aws/app_instance_id/something <br />
	#### *IR AWS : https://tivo.okta.com/home/amazon_aws/0oa6xpru465nLsUoY1t7/272* <br />
	#### *TRA AWS : https://tivo.okta.com/home/amazon_aws/0oa63inc17GkGaXod1t7/272* <br />
- okta_username - use this username to authenticate
- preferred_mfa_type - automatically select a particular  device when prompted for MFA:
  - push - Okta Verify App push
  - token:software:totp - OTP using the Okta Verify App
  - call - OTP via Voice call
  - sms - OTP via SMS message
- resolve_aws_alias - y or n. If yes, gimme-aws-creds will try to resolve AWS account ids with respective alias names (default: n). This option can also be set interactively in the command line using `-r` or `--resolve parameter`
- remember_device - y or n. If yes, the MFA device will be remembered by Okta service for a limited time. This option can also be set interactively in the command line using `-m` or `--remember-device`

## Usage

**If you are not using gimme-creds-lambda nor using appurl settings, make sure you set the OKTA_API_KEY environment variable.**

After running --configure, just run gimme-aws-creds. You will be prompted for the necessary information.

```bash
$ ~/anaconda3/envs/py36/bin/gimme-aws-creds --configure --profile IR
If you'd like to assign the Okta configuration to a specific profile
instead of to the default profile, specify the name of the profile.
This is optional.
Okta Configuration Profile Name [IR]:
Enter the Okta URL for your organization. This is https://something.okta[preview].com
Okta URL for your organization: *https://tivo.okta.com*
Enter the URL for the gimme-creds-server or 'internal' for handling Okta APIs locally.
URL for gimme-creds-server [appurl]:
Enter the application link. This is https://something.okta[preview].com/home/amazon_aws/<app_id>/something
Application url: https://tivo.okta.com/home/amazon_aws/0oa6xpru465nLsUoY1t7/272
Do you want to write the temporary AWS to ~/.aws/credentials?
If no, the credentials will be written to stdout.
Please answer y or n.
Write AWS Credentials [n]: y
Do you want to resolve aws account id to aws alias ?
Please answer y or n.
Resolve AWS alias [n]: y
Enter the ARN for the AWS role you want credentials for. 'all' will retrieve all roles.
This is optional, you can select the role when you run the CLI.
AWS Role ARN: all
If you'd like to set your okta username in the config file, specify the username
.This is optional.
Okta User Name: lkotipibasireddy
If you'd like to set the default session duration, specify it (in seconds).
This is optional.
AWS Default Session Duration [3600]: 43200
If you'd like to set a preferred device type to use for MFA, enter it here.
This is optional. valid devices types:[sms, call, push, token, token:software:totp]
Preferred MFA Device Type: push
Do you want the MFA device be remembered?
Please answer y or n.
Remember device [n]: y
The AWS credential profile defines which profile is used to store the temp AWS creds.
If set to 'role' then a new profile will be created matching the role name assumed by the user.
If set to 'default' then the temp creds will be stored in the default profile
If set to any other value, the name of the profile will match that value.
AWS Credential Profile [role]: IR

$ ~/anaconda3/envs/py36/bin/gimme-aws-creds --register_device
Using password from keyring for lkotipibasireddy
Password for lkotipibasireddy:
Do you want to save this password in the keyring? (y/n) n
Multi-factor Authentication required.
Okta Verify App: SmartPhone_Android: LG-H918 selected
Okta Verify push sent...
Device token saved!

$ ~/anaconda3/envs/py36/bin/gimme-aws-creds
Using password from keyring for lkotipibasireddy
Password for lkotipibasireddy:
Password for lkotipibasireddy:
Do you want to save this password in the keyring? (y/n) y
Password for lkotipibasireddy saved in keyring.
Multi-factor Authentication required.
Okta Verify App: SmartPhone_Android: LG-H918 selected
Okta Verify push sent...
Detected single role: arn:aws:iam::121868943636:role/Admin
export AWS_ACCESS_KEY_ID=ASIARYX7UQEKPOPWARN4
export AWS_SECRET_ACCESS_KEY=/xUSw2HyfH/dnSBpFCeoCIfASzvhgs9b8ziLiX3Z
export AWS_SESSION_TOKEN=FQoGZXIvYXdzEB0aDIog1NudD2NYoBzwZSLCAlzhL1qpg01XwvXWnMpo9UamDQYjutVxnU3JgcwgjAQ6pJFCFKc90R5L6+e8C+JeV6cpSYi3ccjBLO0xjcjw4p4dIFI9tvfP77vm8n0TRjdxKvM8bIeY2HdjhmTqYTUSsp4CEoHO6pfoZpda1vLgyCqXijZNw6uCowMoeHwbJtA0ClS55/tX97uLFSb+Ss1htSYzkEazvBv5/RkG3RiAo5V0vw2ci/GBMFbHGkVZWk5mal7Cp7OsWv3D0VMkAbEm04wIjpa0WmUPFD2qnB3PddgA+zVd7t9KKFlOTZWb0fdW+7LhT66lAaeWO5rN7H7VXGKYdjznjtVSIfDBAz29S4K7a9tyloTLOUrGP8qrDTwfg62dQIlkhKWGUyyk0892hu+/Pyqv0a1CbdqmkzzESu3/4zEKhWXKtkjzEsZUMvqQVTkolrWO6QU=
export AWS_SECURITY_TOKEN=FQoGZXIvYXdzEB0aDIog1NudD2NYoBzwZSLCAlzhL1qpg01XwvXWnMpo9UamDQYjutVxnU3JgcwgjAQ6pJFCFKc90R5L6+e8C+JeV6cpSYi3ccjBLO0xjcjw4p4dIFI9tvfP77vm8n0TRjdxKvM8bIeY2HdjhmTqYTUSsp4CEoHO6pfoZpda1vLgyCqXijZNw6uCowMoeHwbJtA0ClS55/tX97uLFSb+Ss1htSYzkEazvBv5/RkG3RiAo5V0vw2ci/GBMFbHGkVZWk5mal7Cp7OsWv3D0VMkAbEm04wIjpa0WmUPFD2qnB3PddgA+zVd7t9KKFlOTZWb0fdW+7LhT66lAaeWO5rN7H7VXGKYdjznjtVSIfDBAz29S4K7a9tyloTLOUrGP8qrDTwfg62dQIlkhKWGUyyk0892hu+/Pyqv0a1CbdqmkzzESu3/4zEKhWXKtkjzEsZUMvqQVTkolrWO6QU=
```

You can run a specific configuration profile with the `--profile` parameter:

```bash
$  ~/anaconda3/envs/py36/bin/gimme-aws-creds --configure TRA
```

The username and password you are prompted for are the ones you login to Okta with. You can predefine your username by setting the `OKTA_USERNAME` environment variable or using the `-u username` parameter.

If you have not configured an Okta App or Role, you will prompted to select one.

If all goes well you will get your temporary AWS access, secret key and token, these will either be written to stdout or `~/.aws/credentials`.

You can always run `gimme-aws-creds --help` for all the available options.

Alternatively, you can overwrite values in the config section with environment variables for instances where say you may want to change the duration of your token. 
A list of values of to change with environment variables are: `'OKTA_AUTH_SERVER', 'CLIENT_ID','OKTA_USERNAME', 'AWS_DEFAULT_DURATION'`. 

Example: `CLIENT_ID='foobar' AWS_DEFAULT_DURATION=12345 gimme-aws-creds`

For changing variables outside of this, you'd need to create a separate profile altogether with `gimme-aws-creds --configure --profile profileName`

### Viewing Profiles
Run `gimme-aws-creds --list-profiles` will go to your okta config file and print out all profiles created and their settings. 


