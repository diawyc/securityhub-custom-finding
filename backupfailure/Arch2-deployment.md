#  Deployment Process 部署说明

## Step 1Member accounts deployment
Use cloudformation template [Arch2-memberaccounts.yaml](Arch2-memberaccounts.yaml) in your organization's management account to create a stacksets.
Choose the members accounts and regions.
The parameter is the arn of the target Eventbridge event bus, to get it you may run below CLI in the securityhub's delegated admin account.
Copy the output of the CLI command into your cloudformation stacksets paramer EBARN

```
region=eu-west-2
```
```
aws events list-event-buses --region=$region --output text --query "EventBuses[*].Arn"
```
## Step 2 Target Account
Recommand to use securityhub's delegated admin account as the targe account in which to deploy the lambda

### Set Parameter 参数设置
```
region=us-east-1
function='backup-siem-alert'
lambdapolicy='lambda-backup-sechub-policy'
rolename='lambda-backup-sechub'
rulename='backup-lambda-sechub'
```

### Create IAM role for lambda
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

### Create Lambda & Eventbridge
```
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
```
## China Region 中国区
For step1 you need to only choose one region each time, totally rune twice the cloudformation template.
For step2 you can only run once, the same command as SingleAccount Multiple region one, please follow the steps in [SingleAccount- deployment.md](SingleAccount- deployment.md)
