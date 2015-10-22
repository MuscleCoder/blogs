# AWS note

## Crate multiple profile for aws

---
1. aws configure --proifle $proifle_name
2. input all the infomation you are required
3. in ~/.aws/config you will see the config profile you just create

```
[default]
region = ******
aws_access_key_id = ******
aws_secret_access_key = ******

[profile $proifle_name]
region = ******
aws_access_key_id = ******
aws_secret_access_key = ******
```

NOTE: you can just edit the ~/.aws/config file with information.



[Instance Metadata and User Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)

