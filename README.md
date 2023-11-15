<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Usage

Creates a Kinesis Data Firehose Delivery Stream that retrieves records from a Kinesis Data Stream and delivers them to a S3 Bucket.

```hcl

module "kinesis_stream" {
  source = "dod-iac/kinesis-stream/aws"

  name = format("app-%s-%s", var.application, var.environment)
  tags = {
    Application = var.application
    Environment = var.environment
    Automation  = "Terraform"
  }
}

module "kinesis_firehose_s3_bucket" {
  source  = "dod-iac/kinesis-firehose-s3-bucket/aws"

  name = format("app-%s-firehose-%s", var.application, var.environment)

  kinesis_stream_arn = module.kinesis_stream.arn
  kinesis_role_name = format("app-%s-firehose-source-%s", var.application, var.environment)

  s3_bucket_arn = var.aws_s3_bucket_destination
  s3_role_name = format("app-%s-firehose-destination-%s", var.application, var.environment)

  tags = {
    Application = var.application
    Environment = var.environment
    Automation  = "Terraform"
  }
}
```

Creates a Kinesis Data Firehose Delivery Stream that retrieves records from an encrypted Kinesis Data Stream and delivers them to a S3 Bucket encrypted at-rest.

```hcl

module "kinesis_kms_key" {
  source = "dod-iac/kinesis-kms-key/aws"

  name = format("alias/app-%s-kinesis-%s", var.application, var.environment)
  description = format("A KMS key used to encrypt Kinesis stream records at rest for %s:%s.", var.application, var.environment)
  tags = {
    Application = var.application
    Environment = var.environment
    Automation  = "Terraform"
  }
}

module "kinesis_stream" {
  source = "dod-iac/kinesis-stream/aws"

  name = format("app-%s-%s", var.application, var.environment)
  kms_key_id = module.kinesis_kms_key.aws_kms_key_arn
  tags = {
    Application = var.application
    Environment = var.environment
    Automation  = "Terraform"
  }
}

module "kinesis_firehose_s3_kms_key" {
  source  = "dod-iac/s3-kms-key/aws"

  name = format("alias/app-%s-firehose-destination-s3-%s", var.application, var.environment)
  description = format(
    "A KMS key used by AWS Kinesis Data Firehose Delivery Stream to encrypt objects at rest in S3 for %s:%s",
    var.application,
    var.environment
  )

  # To avoid a circular dependency, format the role ARN rather than use
  # output from the following kinesis_firehose_s3_bucket module.
  principals = [format("arn:%s:iam::%s:role/app-%s-firehose-destination-s3-%s",
    data.aws_partition.current.partition,
    data.aws_caller_identity.current.account_id,
    var.application,
    var.environment
  )]

  tags = {
    Application = var.application
    Environment = var.environment
    Automation  = "Terraform"
  }

}

module "kinesis_firehose_s3_bucket" {
  source  = "dod-iac/kinesis-firehose-s3-bucket/aws"

  name = format("app-%s-firehose-%s", var.application, var.environment)

  kinesis_stream_arn = module.kinesis_stream.arn
  kinesis_role_name = format("app-%s-firehose-source-kinesis-%s", var.application, var.environment)

  s3_bucket_arn = var.aws_s3_bucket_destination
  s3_prefix = "data/"
  s3_role_name = format("app-%s-firehose-destination-s3-%s", var.application, var.environment)
  s3_kms_key_arn = module.kinesis_firehose_s3_kms_key.aws_kms_key_arn

  tags = {
    Application = var.application
    Environment = var.environment
    Automation  = "Terraform"
  }
}
```

## Terraform Version

Terraform 0.13. Pin module version to ~> 1.0.0 . Submit pull-requests to master branch.

Terraform 0.11 and 0.12 are not supported.

## License

This project constitutes a work of the United States Government and is not subject to domestic copyright protection under 17 USC § 105.  However, because the project utilizes code licensed from contributors and other third parties, it therefore is licensed under the MIT License.  See LICENSE file for more information.

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.6 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | ~> 5 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | ~> 5 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [aws_iam_policy.kinesis_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy) | resource |
| [aws_iam_policy.s3_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy) | resource |
| [aws_iam_role.kinesis_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) | resource |
| [aws_iam_role.s3_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role) | resource |
| [aws_iam_role_policy_attachment.kinesis_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment) | resource |
| [aws_iam_role_policy_attachment.s3_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment) | resource |
| [aws_kinesis_firehose_delivery_stream.main](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/kinesis_firehose_delivery_stream) | resource |
| [aws_caller_identity.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/caller_identity) | data source |
| [aws_iam_policy_document.assume_role_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) | data source |
| [aws_iam_policy_document.kinesis_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) | data source |
| [aws_iam_policy_document.s3_role](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document) | data source |
| [aws_partition.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/partition) | data source |
| [aws_region.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_cloudwatch_log_group_name"></a> [cloudwatch\_log\_group\_name](#input\_cloudwatch\_log\_group\_name) | The CloudWatch Logs group name for logging.  Defaults to "/aws/kinesisfirehose/[NAME]" | `string` | `""` | no |
| <a name="input_cloudwatch_log_stream_name"></a> [cloudwatch\_log\_stream\_name](#input\_cloudwatch\_log\_stream\_name) | The CloudWatch Logs stream name for logging. | `string` | `"S3Delivery"` | no |
| <a name="input_cloudwatch_logging_enabled"></a> [cloudwatch\_logging\_enabled](#input\_cloudwatch\_logging\_enabled) | Enables or disables the logging to Cloudwatch Logs. | `bool` | `false` | no |
| <a name="input_database_name"></a> [database\_name](#input\_database\_name) | Specifies the name of the AWS Glue database that contains the schema for the output data. Required if using s3\_output\_data\_format\_conversion. | `string` | `null` | no |
| <a name="input_kinesis_role_name"></a> [kinesis\_role\_name](#input\_kinesis\_role\_name) | The name of the AWS IAM Role for reading records from the source AWS Kinesis Stream. | `string` | n/a | yes |
| <a name="input_kinesis_role_policy_document"></a> [kinesis\_role\_policy\_document](#input\_kinesis\_role\_policy\_document) | The contents of the IAM policy attached to the IAM role used by the Kinesis Data Firehose Delivery Stream to read records from the source AWS Kinesis Stream.  If not defined, then creates a default policy. | `string` | `""` | no |
| <a name="input_kinesis_role_policy_name"></a> [kinesis\_role\_policy\_name](#input\_kinesis\_role\_policy\_name) | The name of the IAM policy attached to the IAM Role used by the Kinesis Data Firehose Delivery Stream to read records from the source AWS Kinesis Stream.  If not defined, then uses the value of the "kinesis\_role\_name". | `string` | `""` | no |
| <a name="input_kinesis_stream_arn"></a> [kinesis\_stream\_arn](#input\_kinesis\_stream\_arn) | The AWS Kinesis Stream used as the source of the AWS Kinesis Data Firehose Delivery Stream. | `string` | n/a | yes |
| <a name="input_name"></a> [name](#input\_name) | A name to identify the AWS Kinesis Data Firehose Delivery Stream. This is unique to the AWS account and region the stream is created in. | `string` | n/a | yes |
| <a name="input_processors"></a> [processors](#input\_processors) | A list of processors for the AWS Kinesis Data Firehose Delivery Stream. | <pre>list(object({<br>    type = string<br>    parameters = list(object({<br>      key   = string<br>      value = string<br>    }))<br>  }))</pre> | `[]` | no |
| <a name="input_role_arn"></a> [role\_arn](#input\_role\_arn) | The role that Kinesis Data Firehose can use to access AWS Glue. This role must be in the same account you use for Kinesis Data Firehose. Cross-account roles aren't allowed. Required if using s3\_output\_data\_format\_conversion. | `string` | `null` | no |
| <a name="input_s3_bucket_arn"></a> [s3\_bucket\_arn](#input\_s3\_bucket\_arn) | The ARN of the AWS S3 Bucket that receives the records. | `string` | n/a | yes |
| <a name="input_s3_buffer_interval"></a> [s3\_buffer\_interval](#input\_s3\_buffer\_interval) | Buffer incoming data for the specified period of time, in seconds, before delivering it to the destination. | `number` | `300` | no |
| <a name="input_s3_buffer_size"></a> [s3\_buffer\_size](#input\_s3\_buffer\_size) | Buffer incoming data to the specified size, in MBs, before delivering it to the destination | `number` | `5` | no |
| <a name="input_s3_compression_format"></a> [s3\_compression\_format](#input\_s3\_compression\_format) | The compression format. Options: UNCOMPRESSED, GZIP, ZIP, and Snappy. | `string` | `"UNCOMPRESSED"` | no |
| <a name="input_s3_dynamic_partitioning"></a> [s3\_dynamic\_partitioning](#input\_s3\_dynamic\_partitioning) | If true, enable dynamic partitioning on the AWS Kinesis Data Firehose Delivery Stream. | `bool` | `false` | no |
| <a name="input_s3_dynamic_partitioning_retry_duration"></a> [s3\_dynamic\_partitioning\_retry\_duration](#input\_s3\_dynamic\_partitioning\_retry\_duration) | Total amount of seconds Firehose spends on retries. | `number` | `300` | no |
| <a name="input_s3_error_output_prefix"></a> [s3\_error\_output\_prefix](#input\_s3\_error\_output\_prefix) | Prefix added to failed records before writing them to S3. This prefix appears immediately following the bucket name. | `string` | `""` | no |
| <a name="input_s3_kms_key_arn"></a> [s3\_kms\_key\_arn](#input\_s3\_kms\_key\_arn) | The ARN for the customer-managed KMS key to use for encrypt objects at rest in the AWS S3 Bucket. | `string` | `""` | no |
| <a name="input_s3_output_data_format_conversion"></a> [s3\_output\_data\_format\_conversion](#input\_s3\_output\_data\_format\_conversion) | Convert the data to the specified format before writing to S3. | `string` | `null` | no |
| <a name="input_s3_prefix"></a> [s3\_prefix](#input\_s3\_prefix) | An extra S3 Key prefix prepended before the time format prefix of records delivered to the AWS S3 Bucket. | `string` | `""` | no |
| <a name="input_s3_role_name"></a> [s3\_role\_name](#input\_s3\_role\_name) | The name of the AWS IAM Role for delivering files to the destination AWS S3 Bucket. | `string` | n/a | yes |
| <a name="input_s3_role_policy_document"></a> [s3\_role\_policy\_document](#input\_s3\_role\_policy\_document) | The contents of the IAM policy attached to the IAM role used by the Kinesis Data Firehose Delivery Stream for delivering data to the AWS S3 Bucket.  If not defined, then creates the policy based on allowed actions. | `string` | `""` | no |
| <a name="input_s3_role_policy_name"></a> [s3\_role\_policy\_name](#input\_s3\_role\_policy\_name) | The name of the IAM policy attached to the IAM Role used by the Kinesis Data Firehose Delivery Stream.  If not defined, then uses the value of the "s3\_role\_name". | `string` | `""` | no |
| <a name="input_table_name"></a> [table\_name](#input\_table\_name) | Specifies the AWS Glue table that contains the column information that constitutes your data schema. Required if using s3\_output\_data\_format\_conversion. | `string` | `null` | no |
| <a name="input_tags"></a> [tags](#input\_tags) | Tags applied to the AWS Kinesis Data Firehose Delivery Stream. | `map(string)` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_kinesis_firehose_delivery_stream_arn"></a> [kinesis\_firehose\_delivery\_stream\_arn](#output\_kinesis\_firehose\_delivery\_stream\_arn) | The ARN of the Kinesis Data Firehose Delivery Stream |
| <a name="output_kinesis_firehose_delivery_stream_name"></a> [kinesis\_firehose\_delivery\_stream\_name](#output\_kinesis\_firehose\_delivery\_stream\_name) | The name of the Kinesis Data Firehose Delivery Stream |
<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
