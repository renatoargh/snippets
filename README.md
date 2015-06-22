# snippets
General code snippets.

### Getting Amazon S3 bucket size with aws-cli
```shell
aws s3api list-objects --bucket yourBucket --output json --query "[sum(Contents[].Size), length(Contents[])]"
```
### Easy MySQL database backup to Amazon S3
```shell
function bkp {
    mysqldump -u user -h localhost -ppassword $1 | aws s3 cp - s3://yourBucket/`hostname`/$1/`date +"%Y_%m_%d-%H_%M_%S"`.sql;
}
```
OBS.: Unsecure to add credentials on CLI commands, instead add it to env variables

### Getting latest release version of a given GitHub repository
```shell
curl -u yourUser:yourPass https://api.github.com/repos/yourUserName/yourRepo/releases/latest | grep tag_name | grep -o "[0-9]\.[0-9]\.[0-9]\{1,\}"
```
### Easy Amazon EC2 snapshot backup
```shell
aws ec2 create-snapshot --volume-id vol-df123123 --description "`date +"%Y_%m_%d-%H_%M_%S"` - Backup"
```

### Getting MySQL database size in MB
```sql
SELECT table_schema "Data Base Name",
    sum( data_length + index_length ) / 1024 / 1024 "Data Base Size in MB",
    sum( data_free )/ 1024 / 1024 "Free Space in MB"
FROM information_schema.TABLES
GROUP BY table_schema;
```

### Prevent accidental shutdowns during SSH sessions
```shell
alias shutdown='echo '\''Shutdown not allowed... You can reboot instead'\'''
```
