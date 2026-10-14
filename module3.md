# Module 3: Securing Access to Cloud Resources

## IAM Fundamentals

### AWS Identity and Access Management (IAM)

**What is IAM?**
IAM is a web service that controls who is **authenticated** (signed in) and **authorized** (has permissions) to use AWS resources.

### What IAM Provides

| Capability         | Question it answers                                                      |
| ------------------ | ------------------------------------------------------------------------ |
| **Authentication** | Who is requesting access to the AWS account and the resources in it?     |
| **Authorization**  | After the requester has been authenticated, what are they allowed to do? |

### IAM Overview

| Component      | Description                                                                                                                  |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **User**       | A person or application that can authenticate with an AWS account.                                                           |
| **Group**      | A collection of IAM users who are granted identical authorization. Lets you manage multiple users with the same permissions. |
| **Role**       | An identity used to grant a temporary set of permissions to make AWS service requests.                                       |
| **IAM policy** | A document that defines which resources can be accessed and the level of access to each resource.                            |

### IAM Terminology

| Term             | Meaning                                                                                                                       |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **IAM entity**   | Used by AWS for authentication (users and roles).                                                                             |
| **IAM identity** | Used to identify and group (users, groups, and roles).                                                                        |
| **IAM resource** | The user, group, role, policy, and identity provider objects that are stored in IAM.                                          |
| **Principal**    | A person or application that uses the AWS account root user, an IAM user, or an IAM role to sign in and make requests to AWS. |

### Requests in IAM

A request is made any time a principal attempts to use the AWS Management Console, the application programming interface (API), or the AWS CLI.

A request includes:

- **Actions or operations**: what the principal wants to perform.
- **Resources**: the object upon which the actions or operations are performed.
- **Principal**: the person or application that sends the request by using a user or role.
- **Environment data**: the IP address, user agent, Secure Sockets Layer (SSL) enabled status, or time of day.
- **Resource data**: data related to the resource being requested.

### Service Endpoints

To connect to an AWS service, you must use the URL of the entry point for that service, known as an **endpoint**.

**Example:** the Amazon Elastic Compute Cloud (Amazon EC2) endpoint in US East (N. Virginia) is:

```text
ec2.us-east-1.amazonaws.com
```

## Authenticating with IAM

### IAM Roles

An IAM role provides **temporary security credentials**.

- A role is **not** uniquely associated with one person.
- A role is often used to **delegate access**.

**Use cases:**

- Applications that run on Amazon Elastic Compute Cloud (Amazon EC2)
- Cross-account access for an IAM user
- Mobile applications

### IAM Credentials for Authentication

Two primary types of credentials are used for authentication:

| Access method           | Credential                                                                        |
| ----------------------- | --------------------------------------------------------------------------------- |
| AWS Management Console  | User name and password                                                            |
| AWS CLI, SDKs, and APIs | AWS access key: a combination of an **access key ID** and a **secret access key** |

### Multi-Factor Authentication (MFA)

MFA adds an **extra layer of protection** on top of a user name and password. A user must provide a second factor (something they have) to sign in, so a stolen password alone is not enough to gain access.

**Examples of MFA devices:**

- **Security keys** (for example, YubiKey, Gemalto)
- **Authenticator applications** (for example, Google Authenticator, Authy)
- **Hardware devices** (for example, a hardware TOTP token)

## Authorizing with IAM

### Principle of least privilege

Grant users only the **minimum permissions** needed to perform their job.

**Benefits:**

- Limits damage from compromised credentials
- Prevents accidental damage (a user can't delete what they can't access)
- Helps meet compliance requirements

### Policies and permissions

There are six types of policies:

| #   | Policy type                                | Description                                                                                                                                  |
| --- | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Identity-based policies**                | Attached to IAM identities (users, groups, roles) and grant permissions to that identity.                                                    |
| 2   | **Resource-based policies**                | Attached to resources and grant permissions to the principal specified in the policy.                                                        |
| 3   | **Permissions boundaries**                 | Define the **maximum** permissions that identity-based policies can grant to an entity. They do **not** grant permissions on their own.      |
| 4   | **AWS Organizations SCPs**                 | Service control policies define the **maximum** permissions for member accounts of an organization or organizational unit (OU).              |
| 5   | **Access control lists (ACLs)**            | Control which principals in other accounts can access the resource to which the ACL is attached.                                             |
| 6   | **Session policies**                       | Used with the AWS CLI or AWS API to create a temporary session for a role or federated user.                                                 |

### Identity-based and resource-based policies

| Policy type    | Question it answers                                  | Attached to                  |
| -------------- | ---------------------------------------------------- | ---------------------------- |
| Identity-based | What does a particular identity have access to?      | A user, group, or role       |
| Resource-based | Who has access to a particular resource?             | A resource (for example, S3) |

The difference between the two is **where they are applied**, which depends on the type of access you are authorizing or restricting.

### Managed and inline IAM policies

**Managed policies** can be attached to multiple identities (users, groups, and roles).

**Features:**

- Reusability
- Central change management
- Versioning and rollback
- The ability to delegate permissions management to other users

**Inline policies** are embedded in a **single** principal entity, such as a user, group, or role.

- Useful for a strict 1:1 relationship between a policy and the entity
