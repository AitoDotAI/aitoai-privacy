# Aito Cloud Architecture

The Aito technical architecture can be split into two distinct parts.

1. The database service, storing and operating on the customer data
2. The self-service platform

The configuration and runtime service is handled in separate ways
between the two.

## 1. The database service architecture

The database service runs exclusively in the AWS eu-west-1 region. All the available
Availability Zones (AZ) are used. While there is no guarantee that a given database instance
will run in a given AZ, the running instances are spread out evenly over the three AZs. The entire architecture is set up using
Cloudformation, and hence non of the infrastructure components are
manually configured. This also holds true for all individual customer
databases. Even though the instantiation happens through the self-service console, the backing system handling the setup is Cloudformation, with separated stacks for every environment.

The overall architecture is best summarised by the following diagram

![Architecture diagram](../images/aitoai-architecture.svg)

Each Aito-database is provisioned as a separate Cloudformation stack. This defines all
the private infrastructure components for a database. These are

* The dedicated EBS (Elastic Block Store), with S3-backing for snapshots
* An EC2-instance
* A Security Group, preventing traffic from other than the configured sources
* A Target group
* An ALB Listener Rule for routing the traffic to the given instance
* A Usage Plan provisioned to API Gateway
* The Kong routing rule, mapping the ALB Listener to the hostname

These components are created on demand, and also deleted on instance deletion, either
immediately, or possibly after a delay (e.g. Usage Plans are retained to keep usage
stats available, while API keys are deleted immediately). The other components are shared between all Aito-databases.

Starting from the user, the components and their relations:

### 1.1 API Gateway, Route 53 and Cloudfront

The customer facing part of the stack consists of an API Gateway (AGW) provided JSON Query API.
Due to the way AGW works, a Cloudfront distribution is used to provide the custom hostname (*.aito.app),
which is configured in the AWS DNS service Route53.

The Cloudformation distribution and AGW are configured to only accept TLS-encrypted traffic. TLS 1.2 is
used for the encryption, which is the latest available version in AGW. All traffic outside this boundary, and
within it is encrypted, and the certificates are assumed to be valid, or connetions will fail.

The certificates
for the traffic are managed through the AWS Certificate Manager. The certificates are generated, stored and
updated by AWS. Aito does not create, store or manage the certifictes, except through the AWS provided UI or
Cloudformation.

The main responsibilities of AGW are:

* User authentication
  * no traffic is passed through which does not use a valid, active API Key stored in AGW
  * API keys are mapped to an environment specific usage plan
* API-call monitoring
  * Usage plans are used to store call statistics for each environment
* Throttling
  * AGW sets an upper limit for the total throughput of all the environments
  * Usage plans are used to set limits for each individual environment, preventing neighboring databases to use more than the configured amount of capacity

### 1.2 VPC Link, Network Loadbalancer and Private VPC

The customer environments are separated from the public internet by hosting all components within a private
VPC (Virtual Private Cloud). The VPC is not accessible from the outside, except a few configured exceptions:

* The normal routing allows traffic to flow into the environments, as in the picture above
* SSH-connections are allowed for Aito employees, having valid AWS-credential, enabled through AWS SSM, and
requiring the session to be secured using a MFA-token.

AWS routing from AGW to a private VPC can only happen over a VPCLink, configured in AGW. The traffic between
AGW and Network Load Balancer over the VPCLink is private.

The traffic from the NLB forward is not encrypted. Using client certificates on the customer environment
is also not supported by intermediate components.

### 1.3 Network routing over Kong and AWS Fargate

The NLB needs to be configured to route traffic to a target group,
and cannot thus be forwarded directly to a load balancer. Instead
the traffic to Aito is routed through a Kong-cluster, running
on AWS Fargate.

The Kong configuration is handled by a dedicated database, running
within the VPC. Routing registration, and rule propagation is handled
through this Kong-config. The overall Kong configuration is done through Cloudformation,
which is used to create the down-the-line load balancers and tie them to Kong services.

Kong binds the hostname (more specifically the host-header) for the database to a
configured Application Loadbalancer. The traffic is forwarded to the corresponding ALB.

The Kong-service is run as Docker containers, which are fetched from an AWS-hosted
ECR-repository owned by the Aito AWS-account.

### 1.4 Loadbalancing and Target Groups

Every database instance is bound to a target group, defining the necessary ports and
parameters for the routing. The Application Loadbalancer proxies the traffic to the
correct Target Group based on the host-header in the requests.

The registration between the target group and the ALB is done through Cloudformation.
The ALB to use is assigned on a round-robin basis (with priority given to the ALB with
the fewest active environments) based on a parameter set during the creation of a given
database instance. The load balancer is stable over the lifetime of any given instance,
in order to guarantee upgrades without any forced downtime.

Every ALB is registered to serve traffic on all three available availability zones. This
prevents failures in individual AZs or Loadbalancers to prevent the usage to databases
running in other network partitions.

### 1.5 Aito-database instance and EBS-storage

Each database is provisioned as a separate Cloudformation Stack, meaning that all the
private components are logically owned by the database-environment. The database instance
is built out of the following components

* A private, encrypted EBS-volume, backed by an S3-snapshot. The snapshot(s) is used for (long-term)
data backup, as well as a means to transfer data between new versions of the database
instance
* A private EC2-host, running AWS Linux, which is used to run the container containing the
database runtime. The host is immutable, with the exception of logs being written to the
disk. Upgrades to any component in the host is handled through creating a new instance.

EBS is encrypted using a shared key, stored in AWS KMS. The data is thus encrypted at rest.

EC2 host configuration is handled by a dual setup of Cloudformation template and Puppet.
The bootstrapping is done using Cloudformation, which sets up the base image to contain
the basic binaries, and bootstrap puppet. Puppet is used to install the runtime environment
and all the configuration related to the Aito-service. The sole sources of binaries are
the AWS-provided default yum-repositories, as well as the yum-repository at puppet.com.
We do strict signature checking, so failing signatures cause the installation to fail, and
thus the service to not reach a running state.

The Aito database software is configured to use the AWS provisioned Usage plan and
API-keys tied to the plan. Each request must contain a valid API-key, as configured
in AGW to be passed through. Keys are cached for 5 minutes at a time.

## 2. The self-service platform (aka. Mission Control)

The self-service platform is used for managing the Aito customer environments, but
it does not store or access the data in the database, with a few exceptions described below.

The self-service (later Mission Control) is running on Heroku, and is located in the Heroku
Europe region. Heroku operates on AWS, but the authentication and authorisation is
completely separated and managed through the heroku dashboard.

The responsibilities of Mission Control are

* Allow users to sign up and create their own database instance
* Invite users and manage groups of users who have access to the database
* Show usage stats to the user, related to their private Aito database instance
* Handle subscriptions, payments and invoicing of the database instances
* Manage the lifecycle and updates of Aito database instances

The Console architecture is described by the following diagram:

![Mission Control Architecture diagram](../images/mission-control.png)

All traffic between the components in the diagram is encrypted and authenticatd.
The components in Mission Control have configured shared secrets. Each pair of
services needing to communicate use a separate pair, so we can verify the origin
of the requests.

The calls to AWS are done using a dedicated AWS profile.

## 2.1 The customer facing user interface (Console)

The Console UI is a Single-Page Application (SPA), running in the browser. The UI has a
dedicated server (Console-server), which provides the UI with the necessary data.

Authentication is handled through an external service, Auth0. The Aito Console does
not store or verify any of the passwords or authentication tokens, but rather rely
on Auth0 for these features.

The authentication and session management is handled in the console-server in combination
with Auth0. A user must log in, and have an active session in order to be considered
authenticated. When the user is identified and authenticated, console-server can perform
API-calls on the behalf of the customer. The API-paths are whitelisted and rewritten
before proxied to the CustomerAPI.

Console does not implement any form of an admin UI, so not even Aito-employees are able
access the customer instances using the Console.

## 2.2 The Admin UI (Dashboard)

Dashboard is the Admin-UI, implemented as a SPA running in the browser. Dashboard is
responsible for doing technical management to Aito-database instances for all customers.
These maintenance tasks include

* version upgrades to customer instances
* tuning technical parameters related to the environments
* seeing usage stats for the instances

Dashboard is separated from Console in order to prevent privilege escalation attacks by
the end-users. Console and Dashboard do not share keys and are not accessible from one
another. Dashboard authentication is handled through requiring a valid Google Login in
the Aito.ai domain for accessing the service.

## 2.3 The CustomerAPI, handling the business rules

CustomerAPI is the software that enforces business rules. It is called by direct API-calls
from Dashboard and Console-server. It issues API-calls to Environment-API. CustomerAPI
and Environment-API share a SQS-queue, with which Environment-API communicates progress
of provisioning database environments back to CustomerAPI.

### 2.3.1 Authorization

The authorization is the first main responsibility of the CustomerAPI. It ties the
user id, as provided by the console-server to the permissions of the various (REST-)APIs
for the customer data. Each API-endpoint can have different authorization rules tied to
them:

* Admin-endpoints are only accessible to the separate Admin UI (Dashboard), described later.
* Owner-endpoints require the current user to be the group owner for the particular instance. I.e.
when a user is the owner of a group, he also has admin privileges to the database instances within
that group
* Group-endpoints are accessible only to group-members within a group. The group memebers are
administered by the Owner.
* Anonymous-access endpoints are accessible to all users. These include APIs for e.g. retrieving Aito-database
versions, the available instance sizes for the subscriptions and other technical values.

All API-calls to the CustomerAPI are protected by a service-specific shared secret, which is
added to the requests during the proxying in Console-server. Hence the user cannot get access
to the secret key. The incoming API-calls with a valid shared secret are assumed to be legitimate,
so the whitelisted values like the user id are treated as valid.

### 2.3.2 Business rules

The second main responsibility of Customer API is implementing the business rules. These include
managing user groups, handle invoicing and payments (using Stripe as the backing service), and the
related management of Aito-databases. Customer API requests AWS resources from the Environment API,
and does not have direct access to the AWS-stack.

Business rules are implemented with the assumption that the
the earlier layers performed Authorization and Authentication, so it's possible to operate in a
trusted setting.

### 2.4 Environment API

The main responsibility of the Environment-API (EnvAPI) is to manage AWS-resources. EnvAPI accepts
calls coming from CustomerAPI and maps the requests to appropriate AWS concepts, and provisions
them to the configured AWS region (eu-west-1). The main interactions to AWS are

* Fetching API-keys for users, based on their usage plans
* Fetching usage stats from Cloudwatch Metrics, and storing them in the local database
* Managing the loadbalancers and select where to create a new environment
* Getting software versions from ECR
* Provisioning the database instances using Cloudformation

EnvAPI is only called by CustomerAPI using synchronous API-calls. For long-running operations
EnvAPI communicates back to CustomerAPI using an SQS-queue. The API-calls are protected by
a shared secret between the CustomerAPI and EnvAPI. All traffic is encrypted using TLS.

EnvAPI issues multiple different calls to AWS, and these are executed using the official AWS
node.js SDK. The authentication is handled by a AWS Access Key-pair, which is tied to a
custom user for this purpose. Authorization is hanled through AWS IAM. The user and the
corresponding access rules are created and maintained using Cloudformation.

