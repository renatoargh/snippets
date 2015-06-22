# snippets
Code snippets

### Getting Amazon S3 bucket size with aws-cli
```bash
aws s3api list-objects --bucket yourBucket --output json --query "[sum(Contents[].Size), length(Contents[])]"
```
