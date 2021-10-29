# Appendix. Operational guildelines for cloud infrastructure

## Account policy

Aito.ai production environment runs on AWS infrastructure.
Production and development accounts are completely separated and independent. Production
and test accounts do not share login accounts, passwords or any encryption or access keys.
Accounts are not linked between the two environments.

Customer data is stored only in the production account, and not transmitted or
analyzed in the test account, without first anonymising the data. This is only done with an
explicit approval by the customer.

## Internal usernames and access management on Aito infrastructure
AWS offers different
means of provisioning and managing the infrastructure, namely the AWS Console for
interactive UI-based usage, and API-based access.

* **API-based access**<br/>
  API-based access is handled either through the AWS-command line tools or some of
  the provided programming interface libraries. The libraries are provided at least for
  Java, node.js and Python. The AWS-command line tools are built using the Python
  libraries. API-access is restricted by an AWS-provided access key, but always requires
  the user to also provide a valid MFA-token, as part of the login process.

* **UI-based access**<br/>
  The AWS console can be used to manage every aspect of the infrastructure. Internally
  the console is built using the javascript libraries, and thus it uses the same API-
  endpoints as are being used by the various programming interfaces. The console access
  is protected by a password, but using an MFA-token for all access is mandatory for all
  users.

All users having console-access to the production, i.e. users having 'PowerUser'-privileges,
are created through the scripted environment. Access to the console is then granted
separately to these users by administrator users.
Administrator and PowerUser access keys are not deemed safe to upload to any external
systems, like an external CI/CD-system or provisioning scripts.

***TODO***: Setup policy for rolling the robot-keys on a regular basis.

### API-based access
API-access is enforced based on AWS access keys. These are generated from the console, and
require a separate access key and a secret access key. The keys are automatically generated
by AWS Identity and Access Management (__IAM__). The access keys for users are not shared,
but generated separately for each user. Access keys are available only at generation or
later to the administrator user. Normal and PowerUsers
cannot later download these keys, but they must instead be recreated.

A set of keys can be generated and revoked individually for each user at any given time.

Programmatic access to AWS is handled in two different ways. Services and servers running
in AWS are granted access based on service roles. A service role is a restricted role,
managed through IAM. The permissions for each role are granted on a need-to-have basis,
and each resource is only allowed roles which are necessary for the functionality. Hence,
each service role gets minimal access rights, and each service only has access to roles
it strictly needs.

Service roles are managed through the scripted environment, and all changes can be
tracked back to a user.
See the section [Server infrastructure](#server-infrastructure) for more details.

### Console access
The access to the UI-console is restricted to Aito-employees. There are no shared
accounts, but every user has a private account, which is terminated as soon as employees
leave Aito.

We enforce a strict policy of two-factor authentication for all users with console access.
Two-factor auth for console access is implemented by AWS, and follows industry best
practices.

AWS accounts are terminated as soon as a user leaves the company.

Should external parties require access to parts of the console or functionality provided
through the console, we restrict the access permissions with a minimal-required-functionality
or provide other means of access, e.g. a programmatic dashboard with only the specific
functionality available, as described in the API-based access section.

## Server infrastructure

The Aito service is built using the Infrastructure-as-code principle, meaning that
all changes to the infrastructure are done through means that are possible to replicate:

1. There are separate, separately managed accounts for production use and test use. These do not
share users.
1. All changes are managed through scripts, and manual changes are avoided where possible
1. All change scripts are version controlled in a private repository
1. All changes are peer-reviewed before they are applied to the mainline code
1. The VCS is access controlled, and every change can be tracked back to an individual developer
1. Changes are applied through standard tooling and tested in a separate test
environment before implemented in production
1. Infrastructure supports rollback
1. Infrastructure code is shared. All developers can see, modify and review it at will.
