A Terraform module that creates IAM role with provided JSON IAM polices documents.


This example creates a role with the name `eg-prod-app` with permission to grant read-write access to S3 bucket,
and gives permission to the entities specified in `principals_arns` to assume the role.

```hcl

  data "aws_iam_policy_document" "resource_full_access" {
    statement {
      sid       = "FullAccess"
      effect    = "Allow"
      resources = ["arn:aws:s3:::bucketname/path/*"]

      actions = [
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket",
        "s3:ListBucketMultipartUploads",
        "s3:GetBucketLocation",
        "s3:AbortMultipartUpload"
      ]
    }
  }

  data "aws_iam_policy_document" "base" {
    statement {
      sid = "BaseAccess"

      actions = [
        "s3:ListBucket",
        "s3:ListBucketVersions"
      ]

      resources = ["arn:aws:s3:::bucketname"]
      effect    = "Allow"
    }
  }

  module "role" {
    source = "github.com/jasonfhill/terraform-aws-iam-role"

    enabled   = true
    namespace = "eg"
    stage     = "prod"
    name      = "app"

    policy_description = "Allow S3 FullAccess"
    role_description   = "IAM role with permissions to perform actions on S3 resources"

    principals = {
      AWS = ["arn:aws:iam::123456789012:role/workers"]
    }

    policy_documents = [
      data.aws_iam_policy_document.resource_full_access.json,
      data.aws_iam_policy_document.base.json
    ]
  }
```