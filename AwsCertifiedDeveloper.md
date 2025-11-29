# aws certified developer – associate 
### Iam user 
**IAM stands for identity and access management.( global service).**
- It’s basically an identity that you create in your AWS account to represent a person or an application that needs to interact with AWS resources.
An IAM User is like a login account inside your AWS account — it has:
- a username
- credentials (password or access keys)
and permissions that control what it can or cannot do.

🧩 What Are Permission Policies in IAM?
A permission policy defines what an IAM user (or role/group) can or cannot do in AWS.
It’s a JSON document that lists:
✅ Actions (what can be done),
🧱 Resources (on which things),
🚫 Effect (Allow or Deny),and sometimes Conditions.

🔐 The Two Types of IAM User Permission Policies
1. Managed Policies
These are predefined or separately created policy documents that you can attach to one or more IAM users, groups, or roles.

Types of Managed Policies:
AWS Managed Policies — created and maintained by AWS.
**Examples:**
- AmazonS3FullAccess
- AmazonEC2ReadOnlyAccess
- AdministratorAccess
Customer Managed Policies — created by you (the user).
You define your own JSON rules for fine-grained access control.