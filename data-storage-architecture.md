# Aito Data Storage Architecture

## The Aito Database and AWS

Aito is essentially a data storage service. The service Aito provides is
storing customer data, analysing it, and providing an API to query the
analysed data. The queries are run using Aito's custom API.

The overall architecture is described in [Aito Cloud Architecture](./aito-cloud-architecture.md)
and will not be described in more detail here.

Data for the database is stored in three different locations:

* AWS S3 during long running file uploads
* The private EBS-volume for an instance
* The AWS-created snapshots of the EBS-volume

### S3 storage

The payload size limit for AWS API Gateway is 10Mb per message. This includes headers and the payload.
In order to allow for upload of bigger sets of data Aito allows users to upload data by using
AWS S3 as an intermediate storage location.

The file uploads are done using a private AWS S3-bucket. The bucket is not accessible except for
configured AWS-users and profiles. The database instance is configured to use an instance profile, which
can access the bucket. The database can create a pre-signed upload URL for the user, which allows the
user to upload data to a given file. The path is namespaced per environment, and the name is a random
UUID. The pre-signed URLs only allow writing, not reading or listing data. Pre-signing is valid for
20 minutes, starting from the time of signing.

The S3-uploaded data is accessed by the database instance, but it does so by using the namespaced path,
so it does not access paths, which are outside this boundary.

The S3 data is purged after 7 days of uploading, after which it is no longer available.

### The EBS-volume

Elastic Block Store (EBS) is used as the storage space for the Aito-database. The disk is intialised
with a filesystem, and used as a local disk for the EC2-instance running the database. The software
container is given access only to the logs- and data-partions of the disk. The EBS is encrypted
using the central key stored in KMS. Hence, all the customer data stored in and Aito-instance is
encrypted at rest.

The Aito software treats the mounted EBS-volume as a local disk, and hence has access to any data
stored on it. Since the volume is created for the particular instance, there is never any
data stored, which is not created either by the database in form of logs or the indexed data.
The total disk usage is thus made up of logs, the customer data in the custom format used by Aito,
and the full index of that given data. The data writes, modify and deletion is handled in
transactional fashion, so modifications either complete and modify the data, or fail, and
no change is implemented. This works on a file-system level, but transactions are not guaranteed
on a service-level. Purging of deleted data is done in asynchronous fashion, so the files and the
underlying inodes are available for some time, until they are purged by the async process. The
delay can be up to some hours from the deletion, but happens immediately in case of a shutdown/restart
of the system.

The EBS volumes used by Aito (and mounted to the EC2-instance) are transient. This means that the
lifecycle of a given EBS-volume is tied to the lifecycle of the EC2-instance. During software
updates the entire instance is re-created, and hence the EBS-volume is also destroyed and recreated
from a snapshot. The snapshots are created during the upgrade- and migration-process.

### Snapshots of EBS-volumes

EBS-snapshots are created, managed and handled by AWS, and the commands are issued using the AWS SDK
(and the underlying API). Aito-employees/software only issues commands for snapshots, but does
not have any direct access to the previous snapshots, besides the normal SDK-access.

EBS-snapshots are encrypted using the same mechanism as the volume itself, so using the KMS-stored
central key. The snapshots also act as backups for data for potential catastrophic failure of the
instance.

EBS-snapshots are purged by a scheduled process, which will also remove snapshots for deleted instances.

### Logging in the Aito-database service

In order to provide an adequate level of service, the database creates logs. The logs have been
separated into categories, based on how long they are stored and accessible.

* API-call logs, containing the metadata of the request, like payload-size, time from request to response
and endpoint used are stored in Papertrail and accessible for searching. The log does not contain any
part of the user data. These are used for analytics purposes.
* Info-level logging about the normal operation is stored for 14 days from creation in AWS Cloudwatch logs.
* Debug-level logging is available on the instance, but not transmitted anywhere. For issue response these
logs can be accessed with the same privileges as the database host can. This is restricted to administrators
of the system. The debug logs can contain parts of the data, stack traces,
fragments of queries and other identifiable information.

Access to logs containing any user data is done on a need-to-basis. In case of system failures, we access
logs for root-cause analysis. The actual customer data is never accessed without explicit permission
by the user.

## Customer self-service UI

The self-service UI (Mission Control) has access to, but does not store any customer data. Access to
the data is available by using the API-keys, which the UI handles while showing them to the end-user.
The access to all API keys is authorized using the normal authorization mechanisms in Mission Control.

Mission Control state, and thus the metadata about the user, is stored in AWS RDS databases running
Postgresql. We maintain a Principle of Minimum Data, so we store only data, which is necessary for
providing the service. Aito does not store any customer passwords of any kind, including the API-keys
used to access the database over the API. User login is managed in Auth0, which also does the customer
authentication. We recommend passwordless accounts using Google- or Github-logins to all our users
as a policy. This reduces the potential for weak passwords further.

API-keys are cached during transmission in EnvAPI to avoid throttling errors in AWS. The caching is done
in-memory, so the keys are never written to disk.

Payment related details are handled exclusively by Stripe. Aito does never have access to the actual
payment/credit card details of the user. This data is handled by Stripe and Mission Control only uses
ids provided by Stripe to access the defused data.

Logs are written to Paper trail, but we maintain a policy of not writing any identifying information
in the logs, including API-keys, or names of customers. The user-names are logged on account create,
but after that we use the assigned identifiers (UUID v4) as the sole id of the customers. Logs are
archived in AWS S3 for long-term storage, as mandated by law in Finland.
