# AWS Cli

If using KMS buckets and wanting to copy accounts, you MUST specify KMS

`aws s3 cp --sse aws:kms s3://seth-us-east-2-bucket/seth/warandpeace.txt s3://seth-us-east-2-bucket --profile=$PROFILE_NAME`

