# Module 5: Protecting Data in Your Application

## 1. Introduction and Overview

### 1.1 Module Welcome

This module covers how to protect data in AWS, both at rest (stored) and in transit (moving).

### 1.2 Introduction Section

The introduction sets up the module's objectives, overview, and the bank business scenario.

### 1.3 Module Objectives

By the end of the module, you should be able to:

- Describe how to protect data at rest and in transit.
- Identify Amazon S3 protection features.
- Encrypt data in Amazon S3.
- Differentiate between client-side encryption (CSE) and server-side encryption (SSE).
- Identify AWS services that help protect your data.

### 1.4 Module Overview

- **Sections:** Protect data at rest, S3 protection features, protection through encryption, protect data in transit, best practices for S3, and additional data protection services.
- **Lab:** Encrypting Data at Rest by Using AWS KMS.
- **Knowledge check** at the end.

### 1.5 Bank Business Scenario

**Characters:** María (security lead) and John (who is unsure about AWS security).

- **Scenario 2:** John supports María's plan to protect the bank's assets from outside attacks, but he is unsure about AWS's built-in security.
- **Scenario 3:** At John's previous employer, a disgruntled employee accessed poorly secured user data. The breach was found only after the employee left, and it cost the company a lot in fines and settlements.
- **Scenario 4:** For John to move to the cloud, he needs to know how user data is protected in AWS. María wants to explain how user data stored in the cloud can be secured.

**Lesson:** The scenario shows why data protection (encryption, access control, logging, and monitoring) matters.

### 1.6 Shared Responsibility Model

- **AWS is responsible for security _of_ the cloud:** foundation services (compute, storage, databases, networking) and global infrastructure (Regions, Availability Zones, Edge Locations).
- **The customer is responsible for security _in_ the cloud:**
  - Customer data
  - Platform, applications, and IAM
  - OS, network, and firewall configuration
  - Client-side encryption and data integrity/authentication
  - Server-side encryption (file system and data)
  - Networking traffic protection

This module covers two customer areas: **client-side encryption and data integrity (including authentication)**, and **server-side encryption**.

---

## 2. Protect Data at Rest

Data at rest is data that is stored, such as in S3 buckets, EBS volumes, or databases.

### 2.1 Why Protect Data at Rest?

- Encryption keeps data secure even if an unauthorized party gets access to it.
- It makes attacks harder, even if an endpoint is compromised.
- Business and compliance rules may require it.

Common threats and how to protect against them:

| Threat                                              | Protection                                                                                                                                      |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Information disclosure**                          | Limit who can access data, use policies, and use encryption for confidential data.                                                              |
| **Data integrity compromise**                       | Restrict who can modify data with resource permissions, use digital signatures and encryption, and restore from backup or a previous S3 object version. |
| **Accidental or malicious deletion**                | Apply the correct permissions and least privilege, and restore from backup or a previous S3 object version.                                     |
| **System, hardware, or software failure / natural disaster** | Restore data from replicas.                                                                                                            |

### 2.2 Data at Rest in Amazon S3

- All S3 resources (buckets, objects, and subresources such as lifecycle and website configuration) are **private by default**.
- Only the resource owner (the AWS account that created the resource) can access it.
- The owner can grant access to others by writing an access policy.
- Bucket policies allow granular access to objects.
- S3 Block Public Access is an extra layer that prevents accidental exposure.
- Consider encrypting data at rest in S3 (for example, with AWS KMS).

### 2.3 Granting Permissions: Identity-Based vs. Resource-Based

| Policy type        | Attached to                                       | Defines                                              | Example                                                          |
| ------------------ | ------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------- |
| **Identity-based** | An IAM user (or role/group)                       | What that user is allowed to do                      | Bob's IAM policy allows GET, PUT, and LIST on Bucket X.          |
| **Resource-based** | A resource such as an S3 bucket, VPC endpoint, or KMS key | What a specified user or account can do with that resource | Bucket X's policy allows Bob GET and LIST, and denies PUT. |

**Cross-account access:**

- Identity-based permissions can use IAM roles.
- Resource-based permissions can grant cross-account access directly on the resource, using a list of AWS account IDs.

**Worked example:**

| Bucket       | Bob's identity policy | Bucket policy                          | Result                                                                          |
| ------------ | --------------------- | -------------------------------------- | ------------------------------------------------------------------------------- |
| **Bucket X** | GET, PUT, LIST allowed | GET and LIST allowed; PUT **denied**  | Bob **cannot PUT** into Bucket X (explicit deny wins).                          |
| **Bucket Y** | Only LIST allowed      | GET and LIST allowed; PUT not mentioned | Bob **can read** objects from Bucket Y, even though his identity policy doesn't explicitly allow GET. |

> **Note:** N/A means the policy doesn't specify that action.

### 2.4 Access Control Lists (ACLs)

- ACLs are a separate permission mechanism, independent of IAM and bucket policies. They can be used alongside them.
- Most modern S3 use cases don't need ACLs.
- AWS recommends **disabling ACLs** unless you need per-object access control.
- With ACLs disabled, the bucket policy is the only audit surface, which makes access easier to review.

### 2.5 Key Takeaways: Protect Data at Rest

- Encryption at rest makes it harder for attackers to compromise data.
- S3 data is private by default and requires AWS credentials.
- S3 has two access control types: identity-based (user-based) and resource-based.

---

## 3. Amazon S3 Protection Features

### 3.1 Amazon S3 Block Public Access

- Helps manage public access to S3 resources.
- By default, new buckets and objects don't allow public access, but policies or ACLs can change that. Block Public Access overrides those policies and ACLs.

**The four settings:**

| Setting                   | What it does                                                                                               |
| ------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **BlockPublicAcls**       | Blocks new operations that make buckets or objects public through ACLs. Existing ACLs are not changed.     |
| **IgnorePublicAcls**      | Ignores all public ACLs on a bucket and its objects.                                                       |
| **BlockPublicPolicy**     | Rejects PUT bucket policy calls that allow public access. Existing policies are not changed.               |
| **RestrictPublicBuckets** | Limits access to buckets with public policies to AWS services and authorized users in the bucket owner's account. |

**How they apply:**

- The four settings are independent and can be combined.
- They can be set at the **access point, bucket, or account** level, but not per object.
- If the levels differ, S3 applies the most restrictive combination.
- When S3 gets a request, it checks for a block setting. If one prohibits the access, the request is rejected.
- The S3 console flags public buckets, shows the source of public access, and warns when a policy or ACL change would make a bucket public.

### 3.2 Amazon S3 Versioning

- Keeps multiple versions of an object in the same bucket, so you can preserve, retrieve, and restore every version.
- **Disabled by default.** You must enable it.

**Recovery:**

- Deleting an object adds a **delete marker**. The older versions remain, and removing the marker restores the object.
- Overwriting creates a new version. The previous version can still be restored.
- The most recent version is returned by default. Older versions are retrieved by specifying the version ID.

**Cost:** Every stored version costs money. Use S3 Lifecycle rules to move or delete old versions.

**Bucket states:**

| State                    | Behavior                                                       |
| ------------------------ | -------------------------------------------------------------- |
| **Unversioned** (default) | Overwrites permanently destroy old data.                      |
| **Versioning-enabled**   | Every change creates a new version.                            |
| **Versioning-suspended** | No new versions are created, but existing versions are kept.   |

> **Important:** Once enabled, versioning cannot be turned off, only suspended.

**MFA Delete:** Requires two forms of authentication to delete a version or change the versioning state. Even if credentials are stolen, the attacker cannot wipe version history without the MFA device.

### 3.3 Amazon S3 Object Lock

- Uses a write-once-read-many (**WORM**) model. Objects can't be changed or deleted for a set period or indefinitely.
- Used for regulatory requirements or as an extra layer of protection.
- Requires versioning, because it locks specific version IDs, not file names.

**Two ways to manage retention:**

| Method               | Description                                                                              | Example                         |
| -------------------- | ---------------------------------------------------------------------------------------- | ------------------------------- |
| **Retention period** | Locked for a fixed time, then expires.                                                   | 7 years for compliance.         |
| **Legal hold**       | Same protection with no expiration. It stays until someone explicitly removes it.        | An ongoing investigation.       |

**Two retention modes:**

| Mode           | Behavior                                                                                                   |
| -------------- | ---------------------------------------------------------------------------------------------------------- |
| **Governance** | Users cannot overwrite or delete a version or change its lock settings, unless they have special permissions. |
| **Compliance** | No one can overwrite or delete a protected version, including the root user.                               |

### 3.4 Key Takeaways: S3 Protection Features

- **Block Public Access** ensures objects never have public access, now or in the future.
- **Versioning** preserves, retrieves, and restores every version of every object.
- **Object Lock** stops an object version from being deleted or overwritten for a fixed period or indefinitely.

---

## 4. Protection Through Encryption

### 4.1 Encryption: What, How, and Why

- Encryption uses an algorithm plus a key to turn plaintext into ciphertext, which can only be read again with the right key.
  - Example: "Hello World!" becomes "1c28df2b595b4e30b7b07500963dc7c".
- A strong algorithm produces ciphertext that cannot be decrypted without the key.
- **Key management is the critical part.** If keys are lost or exposed, the encryption fails.

### 4.2 Comparing Client-Side and Server-Side Encryption

| Aspect               | Client-side encryption (CSE)       | Server-side encryption (SSE)                  |
| -------------------- | ---------------------------------- | --------------------------------------------- |
| **When**             | Data is encrypted before it is sent to AWS. | AWS encrypts data after it receives it. |
| **Who encrypts**     | Your application.                  | AWS.                                          |
| **Who manages keys** | You.                               | AWS, or AWS KMS if you choose it.             |
| **What AWS stores**  | Already-encrypted data.            | Plaintext, which AWS encrypts on write.       |
| **User experience**  | You handle encrypt and decrypt.    | Transparent to the user.                      |

- The two approaches are not exclusive. You can use both on the same data.
- Most AWS services that store customer data offer an SSE option, or encrypt by default. Most of them integrate with AWS KMS.

### 4.3 Client-Side Encryption

- Encryption happens before data goes to AWS. Decryption happens after retrieval.
- S3 stores the encrypted data but does not encrypt or decrypt it.
- **Key options:** a key in AWS KMS, or a key stored in your application.
- **Libraries:** AWS Encryption SDK, Amazon DynamoDB Encryption Client, and Amazon S3 encryption clients.

### 4.4 AWS Server-Side Encryption

- Data is encrypted at its destination by the receiving service.
- Data can come from your data center or from EC2 instances.
- Uploads go over HTTPS. The service endpoint handles encryption and key management.
- For S3, encryption is an optional step you specify when uploading.

### 4.5 Types of S3 Server-Side Encryption

You choose one of three mutually exclusive options based on who manages the keys.

#### SSE-C (customer-provided keys)

- You manage the keys and send one with each request. S3 encrypts and decrypts, but you maintain the keys.
- S3 does not store your key. It stores only a salted HMAC value for validation, which cannot be used to recover the key.
- **Warning:** If you lose the key, you lose the object.

#### SSE-S3 (S3-managed keys)

- AWS manages everything. This is the simplest option.
- Each object gets a unique key. S3 encrypts that object key with a master key that it rotates regularly.
- Uses AES-256.

#### SSE-KMS (AWS KMS keys)

- Similar to SSE-S3, with more benefits and extra charges.
- Separate permissions for using the KMS key, which adds protection against unauthorized access.
- Audit trail in CloudTrail showing when the key was used and by whom.
- You can use customer managed keys or AWS managed keys.
- Uses envelope encryption.

#### Quick comparison

| Feature                  | SSE-C | SSE-S3 | SSE-KMS        |
| ------------------------ | ----- | ------ | -------------- |
| **Who manages keys**     | You   | AWS (S3) | AWS (KMS)    |
| **Key stored by S3**     | No    | Yes    | Yes (in KMS)   |
| **Audit trail**          | No    | No     | Yes (CloudTrail) |
| **Envelope encryption**  | No    | No     | Yes            |
| **Lose key = lose data** | Yes   | No     | No             |

### 4.6 SSE-KMS Encryption Flow (Upload)

1. The user uploads a file to an S3 bucket, to be stored encrypted.
2. S3 requests a data key from AWS KMS.
3. KMS generates a plaintext data key and encrypts a copy of it with the customer managed key.
4. KMS sends both copies of the data key to S3.
5. S3 encrypts the object with the plaintext data key, stores it, and deletes the plaintext key. The encrypted data key stays in the object metadata.

**After upload:** the encrypted file exists, the encrypted data key exists in metadata, and the plaintext key is gone.

### 4.7 SSE-KMS Decryption Flow (Download)

1. The user requests the object.
2. S3 notices it is encrypted.
3. S3 sends the encrypted data key to KMS.
4. KMS decrypts it with the customer managed key. That key never leaves KMS.
5. KMS returns the plaintext data key to S3.
6. S3 decrypts the object, gives it to the user, and deletes the plaintext data key.

### 4.8 AWS Key Management Service (AWS KMS)

A managed service for creating and controlling the keys that encrypt your data.

**Features:**

- Create keys with aliases and descriptions.
- Rotate keys automatically on a schedule.
- Disable or delete keys so no one can use them.
- Import your own keys.

**Security:**

- Uses FIPS 140-2 validated hardware security modules (HSMs).
- Keys never leave KMS.
- Key usage policies control who can use which keys.
- All key activity is logged in CloudTrail, including read-only, management, and cryptographic operations.

**Integration:** Services such as S3, EBS, and RDS use KMS, and they protect their own encryption keys with envelope encryption.

### 4.9 AWS KMS Example with Amazon EBS

- EBS encryption is a simple way to encrypt volumes attached to EC2 instances. You don't need to build your own key management.
- Each volume uses AES-256-XTS, which uses two 256-bit keys (effectively one 512-bit key).
- The data key is encrypted under a KMS key in your account. EBS needs a **grant** on the customer managed key to create and use data keys.

**Steps:**

1. EBS gets an encrypted data key from KMS and stores it with the encrypted data.
2. The EC2 host retrieves the encrypted data key.
3. A call to KMS over TLS decrypts the data key using an HSM.
4. The decrypted key is held in memory to encrypt and decrypt all traffic to and from the volume. EBS keeps the encrypted key for later use.

### 4.10 Key Takeaways: Protection Through Encryption

- AWS supports both CSE (you encrypt before sending) and SSE (AWS encrypts after receiving).
- Three SSE types: **SSE-C** (your keys), **SSE-S3** (S3-managed), and **SSE-KMS** (KMS keys).
- AWS KMS creates and controls the keys used to encrypt your data.

### 4.11 Lab: Encrypting Data at Rest by Using AWS KMS

**Lab tasks:**

1. Create an AWS KMS key.
2. Store an encrypted object in an S3 bucket.
3. Attempt public access to the encrypted object.
4. Attempt signed access to the encrypted object.
5. Monitor KMS activity using CloudTrail.
6. Encrypt the root volume of an existing EC2 instance.
7. Disable the encryption key and observe the effects.

**Duration:** 90 minutes.

**Debrief:** The educator may lead a discussion afterward.

---

## 5. Protect Data in Transit

### 5.1 Why Protect Data in Transit?

- Data in transit is data actively moving between locations, such as over the internet or a private network.
- It is often considered less secure while moving.
- Cloud apps often communicate over public links, so protect both client-to-server and server-to-server traffic.

**Risks:**

- Accidental information disclosure.
- Data integrity compromise.
- Identity compromise, including:
  - **Man-in-the-middle (MITM) attacks:** an attacker sits between two parties to steal or change data, often by exploiting weak protocols. This allows packet sniffing and modification.
  - **Identity spoofing:** pretending to be a trusted party.

### 5.2 Protecting Data in Transit

- Use SSL/TLS endpoints (HTTPS) to upload and download S3 data securely.
- Client-side encryption protects data in transit, because it is encrypted before it leaves your system.
- Limit bucket access to a VPC endpoint by using S3 bucket policies.

**Multi-level approach used by AWS:**

| Level                | Protection                                                                                                          |
| -------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Physical layer**   | Traffic between AWS data centers is encrypted.                                                                      |
| **Network layer**    | Traffic within a VPC, and between peered VPCs across Regions, is encrypted on supported EC2 instance types.         |
| **Application layer** | You choose whether to use encryption, such as TLS. All AWS service endpoints support TLS for HTTPS API requests.   |

### 5.3 Protecting Remote Connections to Servers

#### Windows servers: Remote Desktop Protocol (RDP)

- RDP establishes an underlying SSL/TLS connection by default.
- Issue the server a trusted X.509 certificate to protect against spoofing and MITM.
- Avoid the default self-signed certificates, which are not trusted.

#### Linux servers: Secure Shell (SSH)

- SSH is the preferred way to make administrative connections. It creates a secure channel.
- Use tunneling to protect application sessions in transit (for example, X-Windows over SSH).
- Don't allow the root user to log in over SSH.
- Use SSH key pairs for all users, then deactivate password authentication.
- AWS-provided and most AWS Marketplace AMIs already disallow root SSH login and default to key-pair login with passwords off.

### 5.4 AWS Certificate Manager (ACM)

- Makes it easy to enable SSL/TLS, which helps meet regulatory and compliance requirements for encryption in transit.
- **Public certificates:** ACM handles creating and managing public SSL/TLS certificates for AWS-based websites and apps.
- **Private certificates:** ACM can also issue X.509 certificates that identify internal users, computers, applications, services, servers, and devices.
- Provides a single interface for public and private certificates.
- **Deployment:** Certificates can be used on on-premises resources (such as internal servers and IoT devices) and on AWS resources (such as load balancers, CloudFront distributions, and API Gateway endpoints).
- **Renewals:** ACM renews certificates automatically and deploys them to AWS resources, which reduces outages from expired certificates.

### 5.5 AWS Certificate Manager Private Certificate Authority (ACM Private CA)

- Lets you create private CA hierarchies, including root and subordinate CAs, without running your own on-premises CA.

**Use cases:**

- Encrypted TLS channels.
- Authenticating users, computers, API endpoints, and IoT devices.
- Cryptographic code signing.
- OCSP for certificate revocation status.

**Scope:** ACM Private CA is for building a private PKI inside an organization. Certificates it issues are trusted only inside your organization, not on the internet.

**How it works:**

1. The CA administrator creates a subordinate CA.
2. The subordinate CA sends its certificate request for signing.
3. The on-premises CA or AWS CA signs it.
4. ACM Private CA issues signed certificates to resources.

### 5.6 ACM Private CA Considerations

- **HSMs:** The private CA is protected by HSMs that meet FIPS 140-2 Level 3.
- **Access control:** Use IAM policies to control who can administer the CA.
- **Certificate revocation lists (CRLs):** Published and updated automatically to an S3 bucket. For example, an IoT app can check whether a sensor's certificate is still valid.
- **Audit reports:** Stored in another S3 bucket.
- **Common uses:** Securing load balancers, API endpoints, apps, and internal Wi-Fi. Also issuing certificates to EC2 instances, SNS messages, and IoT devices.
- It's a good replacement for self-signed certificates and supports automation through APIs.

### 5.7 Key Takeaways: Protect Data in Transit

- Use SSL or client-side encryption when running apps in the cloud.
- Use VPC endpoints to limit access to S3 buckets.
- ACM handles creating and managing public SSL/TLS certificates.
- ACM Private CA manages the lifecycle of private certificates centrally and with high availability.

---

## 6. Best Practices to Protect Data in Amazon S3

### 6.1 Presigned URLs

- All S3 objects are private by default, and only the owner has access.
- The owner can share an object by creating a presigned URL with their own credentials. This grants time-limited permission to upload or download.

**Three parameters limit access:**

| Parameter   | Meaning                                         |
| ----------- | ----------------------------------------------- |
| **Bucket**  | The bucket the object is in (or will be in).    |
| **Key**     | The object name.                                |
| **Expires** | How long the URL is valid.                      |

**Use cases:**

- Share a private video from a private bucket for a limited time.
- Let someone without an AWS account upload a file to you.

> **Note:** Authentication and authorization for restricted content must be handled by the application.

### 6.2 Security Considerations and Best Practices (1 of 2)

- Use encryption as an extra access control on top of identity, resource, and network controls. Encrypt at rest and in transit.
- Use bucket policies with IAM policies to protect against unauthorized access, information disclosure, integrity compromise, and deletion.
- Apply least privilege. Example: an application that only uploads should not have read permissions.
- Enable MFA Delete on versioning-enabled buckets so files can't be removed or tampered with.
- Enforce SSE on every PUT request. Requests without SSE are denied.
- Use presigned URLs for anonymous access to restricted S3 content.

### 6.3 Security Considerations and Best Practices (2 of 2)

- Set default encryption on the bucket so all objects are encrypted when stored.
- Use the correct Object Lock retention mode (governance or compliance) when using Object Lock. Either mode can apply to any protected version.
- Use S3 Block Public Access to make sure buckets don't allow public access.
- Upload over SFTP with AWS Transfer for SFTP. This is a fully managed service that moves files over SFTP directly into and out of S3, so you don't need to run SFTP infrastructure.
- Enable S3 Versioning to preserve, retrieve, and restore every version of every object.

---

## 7. Additional Data Protection Services

### 7.1 AWS Secrets Manager

A secure, scalable way to manage access to secrets, such as database credentials, passwords, third-party API keys, and arbitrary text.

- Store and control access centrally through the console, CLI, API, or SDKs.

**Benefits:**

- Removes hard-coded credentials from source code and config files.
- Retrieves secrets programmatically, so people reading the code can't see them.
- Automatic rotation on a schedule, which replaces long-lived secrets with short-lived ones.
- Helps meet regulatory and compliance requirements.
- Audits and monitors the secret lifecycle.

### 7.2 Using Secrets Manager (Architecture)

Shows how to secure database credentials for an app that uses an AWS Lambda function to query an Amazon RDS database.

**Flow:**

1. Clients (mobile apps, websites, services) call a RESTful API on Amazon API Gateway over the internet.
2. API Gateway runs the Lambda function.
3. The Lambda function retrieves the database credentials from Secrets Manager through the API. Secrets Manager decrypts and returns them over a secure channel.
4. The Lambda function connects to the RDS database with those credentials and returns the query results.

**Benefit:** No credentials are hard-coded or passed as environment variables.

### 7.3 Rotating Secrets

- **Supported databases for automatic rotation of passwords:** Amazon Aurora, MySQL, MariaDB, PostgreSQL, and Oracle hosted on AWS.
- Turning on rotation requires administrator permissions.
- Rotation runs through a Lambda function, on a schedule or as needed, with no human involvement.
  - **Built-in** Lambda function for RDS database engine credentials.
  - **Custom** Lambda function for other credentials, such as a SAML login with an expiry.

**Four rotation steps.** Secrets Manager uses staging labels on secret versions:

1. The function contacts the database for new credentials.
2. Secrets Manager stores the new credentials with the `AWSPENDING` label.
3. The new credentials are tested.
4. The new credentials become the default with the `AWSCURRENT` label.

### 7.4 Amazon Macie

A security service that uses machine learning and pattern matching to automatically discover, classify, and protect sensitive data in AWS.

**Detects:**

- Personally identifiable information (PII), such as passport numbers, medical ID numbers, and tax ID numbers.
- Financial information, encryption keys, and credentials.
- Custom data types defined with regular expressions for your organization's unique data.

**Scope:** Currently protects data in Amazon S3 only. Available in most Regions.

**Monitoring:** Dashboards and alerts show data access by analyzing S3 resource-based policies and ACLs. Continuous monitoring generates alerts for unauthorized access or accidental leaks.

**Management:**

- Full control through the Macie API.
- Integrates with AWS Organizations: up to 5,000 Macie accounts per organization.
- Native multi-account management: up to 1,000 member accounts under one Macie administrator account.

### 7.5 Classify Data and Monitor Its Access Permissions

- Macie classifies sensitive and business-critical data at the S3 bucket level.
- It scans supported objects and checks them against sensitive data criteria.
- Jobs can be configured. Example: classify only PDFs, or classify everything except objects with a certain prefix.
- Continuous monitoring is on by default for every bucket in an enabled account.
- Findings are generated in near real time for buckets that are:
  - Public.
  - Not encrypted.
  - Shared or replicated with AWS accounts outside your organization.

### 7.6 Macie Summary

- The console summary shows all S3 buckets that Macie has checked.
- Tracks the number of buckets that are public, unencrypted, or shared outside your account, plus total storage and object count.
- Also shows encryption types (SSE-S3 or SSE-KMS).

### 7.7 Macie Findings

- A sensitive data discovery job scans specific S3 buckets for sensitive data using managed data identifiers and optional custom identifiers.
- **Schedules:** one-time, daily, weekly, or monthly. Jobs can cover all objects or a subset.
- For recurring jobs, Macie tracks bucket changes and evaluates only new or modified objects.
- Findings are listed by type, such as `SensitiveData:S3Object/Financial` or `SensitiveData:S3Object/Credentials`, with severity colors.

### 7.8 Key Takeaways: Additional Data Protection Services

- **Secrets Manager** protects access to applications, services, and IT resources.
- **Macie** uses machine learning and pattern matching to discover and protect sensitive data in AWS.
