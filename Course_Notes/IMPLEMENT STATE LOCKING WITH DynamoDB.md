# IMPLEMENT STATE LOCKING WITH DynamoDB

If the state file is stored locally, locking is enabled by default and you dont have to worry about concurrent runs of Terraform, or race conditions.

(`terraform init -migrate-state` will migrate your remote terraform config to a local state file AND vice-versa, if you have a [backend.tf](http://backend.tf/) file configured)

To enable STATE locking on our remote S3 Bucket, we need to search for "dynamodb" on the AWS site.

DynamoDB is a fast, flexible NoSQL database service that is serverless.

To get started, we click "Create Table" on the DynamoDB page.

```jsx
Table name: S3-state-lock
Partition key: "LockID" (type string)
```

This is the partitions primary key and is a hash value used to retrieve items from the table and allocate data across hosts.

All other options are being left empty for now.

To USE our new database / key, we go back to our '[backend.tf](http://backend.tf/)' file and add:

```jsx
dynamodb_table = "s3-state-lock"
```

Now, trying to launch two 'terraform apply' calls at the same time. One will error out with a "state lock" message:

```jsx
Error: Error acquiring the state lock
Error message: ConditionalCheckFailedException: The conditional request
failed
Lock Info:
ID:        9eaeeae9-603b-a4f3-f9fb-67cff1deddd8
Path:      masterterraform53/s3_backend.tfstate
Operation: OperationTypeApply
Who:       psaumur@Sanctuary-VirtualBox
Version:   1.2.9
Created:   2022-09-09 19:59:48.687859377 +0000 UTC
Info:
```