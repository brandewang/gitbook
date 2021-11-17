# Command

## radosgw-admin

```bash
#列出用户
radosgw-admin user list
#创建用户
radosgw-admin user create --uid username --display-name username
#列出桶
radosgw-admin bucket list
#查看桶状态
radosgw-admin stats --bucket="bucketname"
#删除桶并清理对象
radosgw-admin bucket rm --bucket=“bucketname”  --purge-objects
#更改桶的所有人
radosgw-admin bucket link --uid=“username” --bucket="bucketname" --bucket-id=79659a58-0804-42cf-8bf3-317536c24186.374148.19
----------------------------------
#导出桶所属用户信息
radosgw-admin bucket list|grep -v '\[\|\]'|awk -NF'"' '{print $2}'|sort|xargs -I {} sh -c 'echo -n ",{}:"; radosgw-admin bucket stats --bucket={}|jq .owner|tr -d \"|xargs -I {} radosgw-admin user info --uid={}|jq .keys[0]'

#导出桶的所有人
 radosgw-admin bucket list|awk -NF'"' '{print $2}'|grep -v '^$'|sort|xargs -I {} sh -c 'echo -n {};radosgw-admin bucket stats --bucket={}|grep owner'

#导出所有用户key
radosgw-admin user list|awk -NF'"' '{print $2}'|grep -v '^$'|xargs -I {} radosgw-admin user info --uid={}|jq .keys[0]

#批量删桶
#!/bin/bash
for i in `cat bucket.txt`
do
  echo "---------------------------------"
  echo $i
  echo -n "start:";date
  radosgw-admin bucket stats --bucket=$i|grep -w 'num_objects'
  radosgw-admin bucket rm --bucket=$i  --purge-objects
  echo -n "end:";date
done

#桶的最新文件
#!/bin/bash
user_list=`radosgw-admin user list|awk -NF'"' '{print $2}'|grep -v '^$'`

for user in `echo $user_list`
do
  access_key=`radosgw-admin user info --uid=$user|jq .keys[0].access_key|tr -d \"`
  secret_key=`radosgw-admin user info --uid=$user|jq .keys[0].secret_key|tr -d \"`
  for bucket in `s3cmd ls --access_key=$access_key --secret_key=$secret_key|awk '{print $3}'`
  do
    echo -n "$bucket "
    object_info=`s3cmd ls $bucket --access_key=$access_key --secret_key=$secret_key|sort|tail -n 1|awk '{print $1,$2,$4}'`
    echo $object_info
  done
done

#获取桶的文件数
#!/bin/bash
user_list=`radosgw-admin user list|awk -NF'"' '{print $2}'|grep -v '^$'`

for user in `echo $user_list`
do
  access_key=`radosgw-admin user info --uid=$user|jq .keys[0].access_key|tr -d \"`
  secret_key=`radosgw-admin user info --uid=$user|jq .keys[0].secret_key|tr -d \"`
  for bucket in `s3cmd ls --access_key=$access_key --secret_key=$secret_key|awk '{print $3}'`
  do
    echo -n "$bucket "
    object_num=`s3cmd ls $bucket --access_key=$access_key --secret_key=$secret_key -r |wc -l`
    echo $object_num
  done
done
```

## S3cmd

```bash
#生产默认配置文件
s3cmd --configure
vim ~/.s3cfg
access_key=''
host_base = 127.0.0.1:7480
host_bucket = 127.0.0.1
secret_key = ''

#使用指定配置文件
s3cmd --config=/path/config_file

#列出当前用户的桶
s3cmd ls
#递归列出桶内对象
s3cmd ls s3://bucket_name/ --recursive

#创建桶
s3cmd mb s3://bucket_name
#查看桶信息
s3cmd s3://bucket_name
#查看对象信息
s3cmd s3://bucket_name/object
#递归设置acl
s3cmd setacl --acl-grant  full_control:username s3://bucket_name --recursive
s3cmd setacl --acl-revoke full_control:username s3://bucket_name --recursive

#s3_policy
vim public_policy
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": {"AWS": ["arn:aws:iam:::user/brande"]},
        "Action": "s3:*",
        "Resource": [
            "arn:aws:s3:::brande01",
            "arn:aws:s3:::brande01/*"
        ]
    }]
}
#设置policy
s3cmd setpolicy public_policy s3://brande01
s3cmd delpolicy s3://brande01
```

