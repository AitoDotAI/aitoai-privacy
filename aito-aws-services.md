# AWS-services in use

These are the most important AWS-components in use by Aito. The ones marked
as ***required*** are currently needed regardless of whether there is one
or several environments. If hosting only a single environment, the non-required
are possible to omit.

* API Gateway (required)
  * Internet-facing API-proxy, usage plans and API-keys
* Route53
  * Domain name registration for customer environments
* Cloudfront
  * Provides custom domain names for API GW API-endpoints
* Certificate manager
  * Stores and manages the TLS-certificates for the custom domain name(s)
* Network loadbalancer (required)
  * Required for routing traffic from API GW to a private VPC
* Fargate
  * Routing layer and target for the NLB
* Application Loadbalancer (required)
  * Loadbalancing, autoscaling
* EC2 (required)
  * Hosts the database service software
* EBS (required)
  * Acts as the storage (filesystem) for the EC2-host
* S3 (required)
  * EBS-snapshots, large data-uploads using files rather than direct API access
* AWS KMS
  * Key management service for storing data encryption keys
* Cloudwatch (logs and metrics)
  * Logging and monitoring of the environments
