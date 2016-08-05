MySQL S3 Restore
===============

An Ansible role that installs Amazon AWS Command Line Interface and
restores (imports) individual MySQL database backups from Amazon S3.

Requirements
------------

The role assumes you already have MySQL installed.

Only GPG encrypted backups can currently be restored, you will need to have created 
the GPG credentials for this too.

It is recommended that you create a user with locked down permissions.
Below is a policy named `AmazonS3ReadAccess-[bucket-name]`
that can be used to provide limited (create/list/put) access to the
bucket `[bucket-name]`.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [ "s3:ListBucket" ],
                "Resource": [ "arn:aws:s3:::[bucket-name]" ]
            },
            {
                "Effect": "Allow",
                "Action": [ "s3:GetObject" ],
                "Resource": [ "arn:aws:s3:::[bucket-name]/*" ]
            }
        ]
    }

Additionally, you may wish to consider bucket versioning and lifecycles
within S3.

Role Variables
--------------

Available variables are listed below, along with default values (see 
`defaults/main.yml`):

    mysql_restore_name: "mysql-s3-restore"

Name used to identify this role, used for default directory, file and aws profile naming.

    mysql_restore_aws_profile: "{{ mysql_restore_name }}"

For separation we create a new AWS profile for this script context, you
could set this to `"default"` to ignore profiles.

    mysql_restore_aws_access_key: False
    
Your Amazon AWS access key.

    mysql_restore_aws_secret_key: False
    
Your Amazon AWS secret key.

    mysql_restore_aws_region: eu-west-1
    
Region name where the S3 bucket is located.

    mysql_restore_aws_format: text

Output format from the AWS CLI.
    
    mysql_restore_dir: "~/{{ mysql_restore_name }}"
    
Directory used to store downloaded SQL backups. 

    mysql_restore_system_user: root

User that will own the AWS CLI profile.
        
    mysql_restore_innodb_row_format: False
    
Use a ROW_FORMAT value to attempt setting the ROW_FORMAT for any CREATE TABLE statement. 
To enable compression across all tables, you could set this to "COMPRESSED". Please note, 
this would require you to have `innodb_file_format=Barracuda` in your my.cnf.
    
    mysql_restore_remove_imported: True

By default, the original downloaded file and uncompressed versions are deleted after they 
are processed. Using `False` will keep these files in place.

    mysql_restore_databases: []
    
Optional list of databases you wish to restore, specified without the extension. 
If no databases are deinfed, every database in the S3 bucket will be downloaded.
    
    mysql_restore_s3_bucket: False
    mysql_restore_s3_bucket_parent: False
    
Path to your S3 backups bucket. Only set one of these values. 
If you set the s3_backup_parent_url it will expect this to contain a collection of 
sub folders, e.g. daily backups, and will choose the latest (last in the list). 
If you set s3_backup_url this should be a specific backup folder. It assumes the 
backup folder contains one file per database, with the filename as the database name, 
compressed with GZ and encrypted with GPG. Each filename should be of the format databasename.sql.gz.gpg 
    
    mysql_restore_s3_backup_extension: ".sql.gz.gpg"
    
Extension used by the backups stored in s3.
    
    mysql_restore_gpg_secret_key: False
    
GPG secret key that the backups are encrypted for.
    
    mysql_restore_gpg_secret_dest: "~/{{ mysql_restore_name }}-gpg.asc"

Location used to store the GPG secret key.

Dependencies
------------

- [memiah.aws-cli](https://galaxy.ansible.com/memiah/aws-cli/)

Example Playbook
----------------

    - hosts: mysql-servers
      vars_files:
        - vars/main.yml
      roles:
         - memiah.mysql-s3-restore

*Inside `vars/main.yml`*:

    mysql_restore_aws_access_key: "access_key_here"
    mysql_restore_aws_secret_key: "secret_key_here"
    mysql_restore_aws_region: eu-west-1
    mysql_restore_innodb_row_format: COMPRESSED
    mysql_restore_s3_bucket_parent: my-bucket/mysql-backups
    mysql_restore_gpg_secret_key: |
      -----BEGIN PGP PRIVATE KEY BLOCK-----
      Version: GnuPG/MacGPG2 v2.0.22 (Darwin)
      Comment: GPGTools - https://gpgtools.org
      
      [...]
      -----END PGP PUBLIC KEY BLOCK-----
    
License
-------

MIT / BSD

Author Information
------------------

This role was created in 2016 by [Memiah Limited](https://github.com/memiah).
