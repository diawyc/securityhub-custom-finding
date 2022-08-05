#Architecture
![This is an image](/architecture.png)
Result Finding Compare with Inspector in Securityhub
![This is an image](/architecture.png)



# Single Account Multiple Regions Deployment Process 一个AWS账号内多区域部署说明
### Prerequisites 前提条件
Securityhub Enabled and Aggregated Region is set 开启Securityhub并且设定好聚合region
See https://github.com/jessicawyc/aws-enable-ess

请下载所有文件到本地CLI运行目录 Download all the related files from the folder into your local CLI folder,please
### Set Parameter 参数设置
```
regions=($(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text))
function='backup-siem-alert'
lambdapolicy='lambda-backup-siem-policy'
rolename='lambda-backup-siem'
rulename='backup-lambda-sechub'
```
(optional)可检查一下regions里的地区是否是想要部署的 double check the region list
```
echo $regions $function $lambdapolicy $rolename $rulename
```
### Create IAM role 
```
rolearn=$(aws iam create-role --role-name $rolename --assume-role-policy-document file://trust-lambda.json --query 'Role.Arn' --output text)
aws iam put-role-policy --role-name=$rolename --policy-name $lambdapolicy --policy-document file://lambdapolicy.json
```
(optional)Double check the role status 检查role是否正常创建
```
echo $rolearn
```
(optional)if the rolearn is not right,you may use below command to try again.
```
rolearn=$(aws iam get-role   --role-name $rolename --query 'Role.Arn' --output text)
```


## Create Lambda & Eventbridge in each region
```
for region in $regions; do
echo $region
lambdaarn=$(aws lambda create-function \
    --function-name $function \
    --runtime python3.9 \
    --zip-file fileb://index.zip \
    --handler index.lambda_handler \
    --role $rolearn --region=$region --no-cli-pager --query 'FunctionArn' --output text)
echo $lambdaarn
rulearn=$(aws events put-rule \
--name $rulename \
--event-pattern "{\"source\": [\"aws.backup\"],\"detail-type\": [\"Backup Job State Change\"]}" \
--query 'RuleArn' --output text \
--region=$region)
echo $rulearn
aws lambda add-permission \
--function-name $function \
--statement-id eb-rule \
--action 'lambda:InvokeFunction' \
--principal events.amazonaws.com \
--source-arn $rulearn --region=$region
aws events put-targets --rule $rulename  --targets "Id"="1","Arn"=$lambdaarn --region=$region
done
```

