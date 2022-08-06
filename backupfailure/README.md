#Overall Architecture
## Prerequisites 前提条件
Securityhub Enabled and Aggregated Region is set with delegated admin account within Orgnization 开启Securityhub并且设定好聚合region,组织内指定管理员账号
See https://github.com/jessicawyc/aws-enable-ess

There all 3 types of deployment architectures:
## Single Account Multiple Regions Architecture 一个AWS账号内多区域架构
![type1](Arch-SingleAccount.png)
Deployment process see [SingleAccount](SingleAccount- deployment.md)
## Multiple Accounts with Multiple Regions in one Organization  Architecture 1 组织内多账号多区域架构1
Repeat the SingleAccount deployment in every member account, as securityhub with organization has the aggregation feauture in nature, all the findings can be aggregated in the aggregated region in delegated admin account securityhub.
![type1](Arch-SingleAccount.png)
Deployment process is the easiest one, just run a cloudformation stacksets template in your management account
for Detail stpes please see [Arch1-deployment.md](Arch1-deployment.md)
## Multiple Accounts with Multiple Regions in one Organization  Architecture 2 组织内多账号多区域架构2
Each region in each memeber account alert will be sent to a central Eventbridge eventbus in the securityhub delegated admin account, then there will be only one lambda centrally process all the events,and generate a critical finding in securityhub.
![type1](Arch-SingleAccount.png)
Deployment process detail please see [Arch2-deployment.md](Arch2-deployment.md)


