# snippets
General code snippets.

### How to reset auto increment value for a MySQL table?

```sql
ALTER TABLE tablename AUTO_INCREMENT = 1
```

### How to cleanup MySQL database completely

```sql
SET FOREIGN_KEY_CHECKS = 0;
SET GROUP_CONCAT_MAX_LEN = 32768;
SET @tables = NULL;
SELECT GROUP_CONCAT('`', table_name, '`') INTO @tables FROM information_schema.tables WHERE table_schema = (SELECT DATABASE());
SET @tables = CONCAT('DROP TABLE IF EXISTS ', @tables);
SELECT IFNULL(@tables, 'SELECT 1') INTO @tables;
PREPARE stmt FROM @tables;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;
SET FOREIGN_KEY_CHECKS = 1;
SET @views = NULL;
SELECT GROUP_CONCAT(table_schema, '.', table_name) INTO @views
 FROM information_schema.views
 WHERE table_schema IN (select database());
SET @views = IFNULL(CONCAT('DROP VIEW ', @views), 'SELECT "No Views"');
PREPARE stmt2 FROM @views;
EXECUTE stmt2;
DEALLOCATE PREPARE stmt2;
```

### Getting Amazon S3 bucket size with aws-cli
```shell
aws s3api list-objects --bucket yourBucket --output json --query "[sum(Contents[].Size), length(Contents[])]"
```
### Easy MySQL database backup to Amazon S3
```shell
function bkp {
    mysqldump -u user -h localhost -ppassword --single-transaction $1 | aws s3 cp - s3://yourBucket/`hostname`/$1/`date +"%Y_%m_%d-%H_%M_%S"`.sql;
}
```
OBS.: Unsecure to add credentials on CLI commands, instead add it to env variables

### Getting latest release version of a given GitHub repository
```shell
curl -u yourUser:yourPass https://api.github.com/repos/yourUserName/yourRepo/releases/latest | grep tag_name | awk '{print $2}' | cut -d "\"" -f 2
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
