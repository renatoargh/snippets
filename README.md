# snippets
General code snippets.

### Ensuring that if one value is provided then another is also necessary in JavaScript

```javascript
if ((latitude === null) !== (longitude === null)) {
    throw new Error('Require either both latitude and longitude or neither')
}
```

### How do I get the average from a series of values from stdin?

Say you have the file:
```text
foo 3
bar 4
zaz 9
rebers 10
```

You can get the average value of the second column by using `awk` like this:

```bash
cat file.txt | awk '{ total += $2; count++ } END { print total/count }'
```

### How do I find the whole data for the row with some max value in a column per some group identifier in MySQL?

```sql
SELECT a.*
FROM YourTable a
LEFT OUTER JOIN YourTable b
    ON a.id = b.id AND a.revision < b.revision
WHERE b.id IS NULL;
```

_Taken from StackOverflow: http://stackoverflow.com/a/7745635/91403_

### How do I change MySQL user password?
```sql
SET PASSWORD FOR 'user-name-here'@'hostname-name-here' = PASSWORD('new-password-here');
FLUSH PRIVILEGES;
```

### How do I delete/remove MySQL user?
```sql
DROP USER 'user'@'localhost';
```

### How to reset auto increment value for a MySQL table

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

You may need to drop all events from the database. I am not sure why the following snippet doesn't work
```sql
SET @events = NULL;
SELECT GROUP_CONCAT('`', EVENT_NAME, '`') INTO @events FROM information_schema.events  WHERE EVENT_SCHEMA = (SELECT DATABASE());
SET @events = CONCAT('DROP EVENT IF EXISTS ', @events);
SELECT IFNULL(@events, 'SELECT 1') INTO @events;
select * from @events;
PREPARE stmt2 FROM @events;
EXECUTE stmt2;
DEALLOCATE PREPARE stmt2;
```

### Getting Amazon S3 bucket size with aws-cli
```shell
aws s3api list-objects --bucket yourBucket --output json --query "[sum(Contents[].Size), length(Contents[])]"
```
### Easy MySQL database backup to Amazon S3 with aws-cli and mysqldump
```shell
function bkp {
    /usr/bin/mysqldump -u user -h localhost -ppassword $1 | gzip | /usr/local/bin/aws s3 cp - s3://bucket/backups/`hostname`/$1/`date +"%Y_%m_%d"`/`date +"%Y_%m_%d-%H_%M_%S"`.gz;
}

# or

bkp() {
    /usr/bin/mysqldump -u user -h localhost -ppassword $1 | gzip | /usr/local/bin/aws s3 cp - s3://bucket/backups/`hostname`/$1/`date +"%Y_%m_%d"`/`date +"%Y_%m_%d-%H_%M_%S"`.gz;
}
```
OBS.: It is unsecure to add credentials on CLI commands, instead add it to env variables

### Full MySQL instance backup to Amazon S3 with aws-cli and mysqldump

**backup.sh**
```bash
#!/bin/bash

function bkp {
    echo " > $1"

    [ -f backup.sql ] && rm backup.sql 
    [ -f backup.sql.gz ] && rm backup.sql.gz

    mysqldump --login-path=backup --events --default-character-set=utf8 --result-file=backup.sql --single-transaction $1
    gzip backup.sql 
    aws s3 cp --quiet backup.sql.gz s3://yourBucket/backups/`hostname`/$1/`date +"%Y_%m_%d"`/$1`date +"-%d_%m_%Y-%H_%M_%S"`.gz;

    [ -f backup.sql ] && rm backup.sql 
    [ -f backup.sql.gz ] && rm backup.sql.gz
}

echo Iniciando backup em `date +"%d/%m/%Y %H:%M:%S"`

for db in $(mysql --login-path=backup -e 'show databases' | node filtrarDatabases.js); do
    bkp ${db}
done

echo Backup finalizado  em `date +"%d/%m/%Y %H:%M:%S"`
echo ---------------------------------------
```

**filtrarDatabases.js**
```javascript
var readline = require('readline'),
    rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
        terminal: false
    }),
    ignorar = [
        'Database', 
        'mysql',
        'information_schema', 
        'performance_schema',
        'Warning: Using a password on the command line interface can be insecure.'
    ];

rl.on('line', function(line) {
    line = line.replace(/\+-\|/g, '').trim();

    if(ignorar.indexOf(line) === -1) {
        console.log(line);
    }
});
```

Before running `backup.sh` you will have to execute `mysql_config_editor set --login-path=backup --host=localhost --user=yourMySqlUser --password` (it will then ask for your password). Its just to prevent using password on the command line interface.

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
