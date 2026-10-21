# Protecting Data in Your Application

## Protect Data at Rest

### Why Protect Data at Rest?

Common scenarios and their mitigations:

- **Information disclosure**
  - Limit access to data.
  - Use policies to manage access to resources.
  - Use encryption to protect confidential data.

- **Data integrity compromise**
  - Limit users' ability to modify data.
  - Implement digital signatures and encryption.
  - Restore data from a backup or an object version.

- **Accidental or malicious deletion**
  - Use the correct permissions and the principle of least privilege.
  - Restore data from a backup or an object version.

- **System, hardware, and software availability**
  - Restore data from replicas.

### Data at Rest in Amazon S3

By default, data stored in Amazon S3 is **private** and requires AWS credentials for access.

### Granting Permissions: Identity-Based vs. Resource-Based

| Policy type        | Attached to                     | Indicates                                                       |
| ------------------ | ------------------------------- | --------------------------------------------------------------- |
| **Identity-based** | An AWS IAM user, group, or role | What that identity is permitted to do                           |
| **Resource-based** | A resource (e.g., an S3 bucket) | What a specified principal is permitted to do with the resource |

## Amazon S3 Protection Features

### Amazon S3 Block Public Access

Manages public access to Amazon S3 resources through four settings:

| Setting                   | Effect                                                                       |
| ------------------------- | ---------------------------------------------------------------------------- |
| **BlockPublicAcls**       | Blocks public access granted by new ACLs                                     |
| **IgnorePublicAcls**      | Ignores all public ACLs on a bucket and any objects it contains              |
| **BlockPublicPolicy**     | Blocks public access granted by new public bucket policies                   |
| **RestrictPublicBuckets** | Blocks public and cross-account access granted by any public bucket policies |

### Amazon S3 Versioning

- Creates a new version with every upload.
- Protects against unintended deletion.
- Allows retrieval of deleted objects.

### Amazon S3 Object Lock

- Stores objects using the write-once-read-many (WORM) model.
- Works only in versioned buckets.
- Manages object retention with two modes:

| Mode           | Behavior                                                                        |
| -------------- | ------------------------------------------------------------------------------- |
| **Governance** | Objects can't be overwritten or deleted unless the user has special permissions |
| **Compliance** | Objects can't be overwritten or deleted by anyone                               |

## Protect Through Encryption

### Encryption

**What?**

- Uses an algorithm and a key to convert plaintext data into unreadable data (ciphertext).
- The ciphertext can become readable again only with the right key.

**How?**

- Many encryption algorithms exist, and they use different types of keys.
- A strong algorithm relies on mathematical properties to produce ciphertext that can't be decrypted with any practically available amount of computing power unless you also have the necessary key.

**Why does key management matter?**

- Protecting and managing the keys is a critical part of any encryption solution.

### Client-Side vs. Server-Side Encryption

| Type                             | Where encryption happens                                             | Key points                                                                                                                                                                                             |
| -------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Client-side encryption (CSE)** | Locally in your application, before data is submitted to AWS         | Your application decrypts data after receiving it from AWS. Data is stored in encrypted form, with keys and algorithms known only to you. The key can be stored in AWS KMS or within your application. |
| **Server-side encryption (SSE)** | At the destination, by the application or service that receives data | The encryption process is transparent to the user.                                                                                                                                                     |

### Types of S3 Server-Side Encryption

| Type                                | Key points                                                                                    |
| ----------------------------------- | --------------------------------------------------------------------------------------------- |
| **SSE-C** (customer-provided keys)  | You retain control of the keys. Amazon S3 doesn't store the encryption keys that you provide. |
| **SSE-S3** (Amazon S3 managed keys) | AWS manages the keys. The encrypted data and keys are stored on separate hosts.               |
| **SSE-KMS** (AWS KMS keys)          | AWS manages the keys. An envelope key is used for added protection.                           |

### Encryption Overview

1. You request to upload a file and store it as an encrypted object in an S3 bucket.
2. Amazon S3 requests a data key from AWS KMS to use to encrypt the file.
3. AWS KMS generates a plaintext data key and encrypts the data key by using the customer managed key.
4. AWS KMS sends both copies of the data key to Amazon S3.
5. Amazon S3 encrypts the object by using the plaintext data key, stores the object, and then deletes the plaintext data key. The encrypted key is kept in the object metadata.

### Decryption Overview

1. You request to open the object.
2. Amazon S3 notices that the requested object is encrypted.
3. Amazon S3 sends the encrypted copy of the data key that the object is encrypted with to AWS KMS.
4. AWS KMS decrypts the data key by using the customer managed key, which never leaves the AWS KMS service.
5. AWS KMS sends the plaintext data key back to Amazon S3.
6. Amazon S3 decrypts the ciphertext of the data object, allows you to open the object, and deletes the plaintext copy of the data key.

### AWS Key Management Service (AWS KMS)

- Lets you create and manage cryptographic keys.
- Uses hardware security modules (HSMs) to protect your keys.
- Is integrated with other AWS services.
- Lets you set usage policies that determine which users can use keys.

## Protect Data in Transit

### Why Protect Data in Transit?

Your communications might go through the public internet. Data in transit is exposed to these risks:

- Accidental information disclosure
- Data integrity compromises
- Identity compromises
- Man-in-the-middle (MITM) attacks
- Identity spoofing

### How to Protect Data in Transit

- Use Secure Sockets Layer (SSL) endpoints over Transport Layer Security (TLS) (HTTPS).
- Use encryption.
- Use an Amazon Virtual Private Cloud (Amazon VPC) endpoint to limit access to your bucket.

### Protecting Remote Connections to Servers

**Windows servers (RDP)**

- Windows Terminal Services in the public cloud usually use the Microsoft Remote Desktop Protocol (RDP).
- RDP connections establish an underlying SSL/TLS connection.
- For better security, issue a trusted X.509 certificate to protect against identity spoofing and MITM attacks.
- Don't use self-signed certificates.

**Linux servers (SSH)**

- Secure Shell (SSH) is the preferred approach for establishing administrative connections to Linux servers.
- SSH is a protocol that, like SSL, provides a secure communications channel between the client and server.

### AWS Certificate Manager (ACM)

- Provides a single interface to manage both public and private certificates.
- Makes it easy to deploy certificates.
- Protects and stores private certificates.
- Minimizes downtime and outages with automatic renewals.

### AWS Certificate Manager Private Certificate Authority (ACM Private CA)

ACM Private CA in action:

1. The CA administrator creates a subordinate CA to issue certificates.
2. The subordinate CA sends the certificate to be signed.
3. The on-premises CA or AWS CA signs the certificate.
4. ACM Private CA issues the signed certificate to resources.

### ACM Private CA Considerations

- ACM Private CA is secured with HSMs.
- CA administrators can control access to the service by using IAM policies.
- ACM Private CA automatically publishes and updates certificate revocation lists to an S3 bucket to help prevent the use of revoked certificates.
- ACM Private CA also creates another S3 bucket that provides the ability to generate audit reports.

## Additional Data Protection Services

### AWS Secrets Manager

- Provides a secure and scalable method for managing access to secrets.
- Can automatically rotate a secret on a schedule that you specify.
- Significantly reduces the risk of compromise.

### Amazon Macie

- Uses machine learning to automatically discover, classify, and protect sensitive data in AWS, such as personally identifiable information (PII), financial information, encryption keys, and credentials.
- Supports custom-defined data types.
- Protects data stored in Amazon S3 only, and is available in most AWS Regions.
- Monitors resource policies and ACLs to protect data stored in Amazon S3.
- Gives you full control of the service through the Macie API set, and lets you centrally manage Macie for multiple accounts.
