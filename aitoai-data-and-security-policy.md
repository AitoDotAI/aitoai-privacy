<head>
<link rel="stylesheet" type="text/css" media="all" href="./aito-platform-and-data.css" />
</head>

<img src="./aito_logo.svg" alt="The Aito logo" width="150px" />

# Aito Data and Security Policy

This is a draft of the Aito Data Policy. It contains a description of principles and
procedures we apply in our daily work.

Aito is a startup, founded in 2018. We operate with a small, dedicated, tech-oriented team.
While we haven't had resources to get an official 3rd party certificate, we take
data security seriously. Our aim is to start a certification process within 2021, to show
our commitment to these principles.

## Aito.ai service description

### Service description
Aito.ai is a software-as-a-service company (__SaaS__), which offers a data storage and
Machine Learning solution for software automation.

Aito runs on cloud infrastructure provided by Amazon Web Services
(__AWS__), and does not operate any own hardware.

The Aito service consists of the following logical components:

1. The Aito Console is a self-service application, where the customer can manage their
subscriptions, handle payment and account details, run data analysis and visualisation,
and use limited query capabilities.
2. The Aito Core instance, which is the custom software that stores the customer data, and
implements the ML-functionality. The instance is accessible only over an API, protected by a
set of API (shared secret) access keys. The API also used when running analysis or queries
through the Console. Customer uploaded data is only
stored in the instance, not in the other parts of the service.


Aito handles all the server infrastructure, encryption certificates, data storage and
backups on behalf of the customer. The service could thus be seen as a turn-key service,
with the
customer being responsible for building the integration through the provided API, and Aito
taking care of all the management and running the service.

## GDPR compliance
Aito operates and is registered as a legal entity in Finland.
The Aito infrastructure is provisioned in the AWS Region **eu-west-1**.

The eu-west-1 data centres physically reside in Dublin, Ireland. Like Finland,
Ireland is also a member of the European Union (EU) and hence we comply to the laws
and regulations mandated by Finland and the EU.

The most prominent requirement is the [General Data Protection Regulation,
the __GDPR__](https://www.eugdpr.org/). Under
the GDPR Aito operates as the Processor of data, whereas our customers are the
Controller. As a data processor we are committed to help our customers comply
with the GDPR regulations. We operate solely in AWS EU-based regions, and AWS
acts as a Processor of the data. As per the
[Data Processing Agreement provided by AWS](https://d1.awsstatic.com/legal/aws-gdpr/AWS_GDPR_DPA.pdf)
AWS does not access or utilize the stored data in any way (§3 of the DPA).

The GDPR applies to all stored data, which can be used to identify individual persons.
The GDPR is not applied in cases where the data is anonymised in a way that prevents
linking it back to individuals. Our customer, in the capacity of
Controller, is ultimately responsible for any data stored into the system.
Aito does not parse or use any semantic information about
the data, and therefore we have no way of distinguishing between compliant and
non-compliant data. We expect our customer to take necessary precautions when parsing,
processing and storing data that is under the GDPR.

In order to comply with the GDPR Aito will enter into a data processing agreement
with our customer before accepting any GDPR regulated data into the Aito service.

## Data ownership and management

### Use of Data
All data stored in the Aito system is owned by the end-user/customer. Aito does not access
or use the data in any way, except what is necessary to provide the service. This means
making sure the data is available to the Aito Core software, occasional backups are
performed, and that the data is deleted if the customer terminates a subscription and
deletes an instance.

Aito employees will not view the customer data, unless explicitly agreed with the customer.
The only exception to this rule is metadata (size of the data, cleaning of temporary files,
query formats and headers), which does not directly reveal any content. Should we need to
access live data, an explicit approval will be requested from the client before we access
or operate on the data.

Anonymised instance logs are stored as required by authorities, and to the extent it is
necessary for providing the service. These logs are anonymous, and contain no customer data.
Logs are retained for 3 months (60 days), after which they are deleted from the
system. Local log dumps are deleted once the purpose for accessing the logs is
no longer valid.

Aito will not use the customer data for any internal or external purpose.
The data is never shared with any external parties or people not employed with Aito.
Data access behaviour and metadata about performance or query characteristics are, however,
monitored and used for improving the level of the service. This data is anonymous and
cannot be used to deduce information about service users.

### Data encryption

Data-at-rest is encrypted at Aito. This guiding principle is followed both for customer data
as well as logs. The data is stored in 3 different phases: live data on EBS-volumes,
volume backups in S3 and log data.

Aito does not implement any custom encryption algorithms, but rather uses industry best
practices encryption, as provided by AWS. Encryption procedures are implemented by the
cloud provider.

Aito does not store encryption keys and certificates in any places besides the default
storage locations in AWS Certificate manager and KMS. The keys are protected by the
same authentication and authorization mechanisms as the rest of the infrastructure, and
access is granted on a need-to basis.

A more detailed explanation of principles and procedures can be found in a [separate
appendix](./data-and-security-policy.appendix/encryption.md).


## General operational guidelines

The operational guidelines are based on industry best practice. The guiding principles are

* Personal accounts for each user
* Access privileges on a need-to basis
* Lest privilege
* Multi-factor authentication
* Completely separated test- and production-environments
* All changes through scripted and version-controlled tooling.
* Peer reviews before applying to test, and testing before production
* Encryption-at-rest where possible
* Separation of customer

A more detailed description can be found in the [Operational guidelines appendix](./data-and-security-policy.appendix/aws-operational.md)
to this document.

## Software development

The software development practices at Aito follow industry best practice. The guiding
principles are

* Shared codebase, and every developer is free to contribute anywhere
* All development through a process of version control. CI, peer-reviews and deployments
through a trusted path
* Scripted and repeatable setup for any tasks, which are performed multiple times
* Design and coding principles follow the [Twelve-factor app guidelines](https://12factor.net)

The detailed guidelines are maintained in a separate Google Document, which contains specific
addresses, repositories and locations. A cleaned version of it can be provided as an appendix,
although the guidelines are considered a working document and get amendments, improvements and
changes as a matter of the working process.

## Customer user accounts

Aito does not store user passwords in any form or in any service. We use an external
service to maintain the accounts, and use the provided API to access the functionality.
Thus, Aito does not have access to neither plain-text nor encrypted passwords created by
the users. We also encourage people to use an existing account through single sign-on
(Google Login, or Github login) rather than creating a separate account for Aito.

Aito instance keys are a logically passwords. They are only stored in and verified against
the original system, which is the AWS API Gateway service. Every authentication and
authorization is verified against the original key, and they are not permanently stored in
any other system. Transient caching is employed for performance. The keys are verified at
multiple stages, making it harder to be able to bypass the security mechanisms in place.

---
<img src="./aito_logo.svg" alt="The Aito logo" width="50px" />
<div class="footer">
<span>Questions, suggestions, comments?</span>
<span>/</span>
<span><a href="https://aito.ai">Aito.ai</a></span>
<span>/</span>
<span><a href="mailto:hello@aito.ai">hello@aito.ai</a></span>
</div>
