#  Deployment Process 部署说明

## Member accounts deployment
Use cloudformation template [Arch2-memberaccounts.yaml](Arch2-memberaccounts.yaml) in your organization's management account to create a stacksets.
Choose the members accounts and regions.
The parameter is the arn of the target Eventbridge event bus, to get it you may run below CLI in the securityhub's delegated admin account.

```
region=eu-west-2
```
```
aws events list-event-buses --region=$region --output text --query "EventBuses[*].Arn"
```
## Target Account
Recommand to use securityhub's delegated admin account as the targe account in which to deploy the lambda

