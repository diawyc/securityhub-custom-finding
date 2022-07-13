# Deployment Process 部署说明

请下载所有文件到本地 Download all the related files here
### Set Parameter参数设置
```
regions=($(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text))
function='rds-replicate-siem'
lambdapolicy='lambda-rds-replicate-siem-policy'
rolename='lambda-rds-replicate-siem'
function='rds-replicate-siem'
rulename='rdsreplicate-lambda'
rolearn=$(aws iam create-role --role-name $rolename --assume-role-policy-document file://trust-lambda.json --query 'Role.Arn' --output text)
aws iam put-role-policy --role-name=$rolename --policy-name $lambdapolicy --policy-document file://lambdapolicy.json
```

## Create Lambda in each region
```
for region in $regions; do
echo $region
lambdaarn=$(aws lambda create-function \
    --function-name $function \
    --runtime python3.9 \
    --zip-file fileb://index.zip \
    --handler index.lambda_handler \
    --role $rolearn --region=$region --no-cli-pager --query 'FunctionArn' --output text)
rulearn=$(aws events put-rule \
--name $rulename \
--event-pattern "{ \"detail\": {\"eventName\": [\"CreateDBInstanceReadReplica\"]}}"  --region=$region)
aws lambda add-permission \
--function-name $function \
--statement-id eb-rule \
--action 'lambda:InvokeFunction' \
--principal events.amazonaws.com \
--source-arn $rulearn --region=$region
aws events put-targets --rule $rulename  --targets "Id"="1","Arn"=$lambdaarn --region=$region
done
```

