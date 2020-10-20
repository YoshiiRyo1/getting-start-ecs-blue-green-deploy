# Step 2
## IAM ロール その2
### ecsTaskExecutionRole
ECS タスク実行用の IAM ロールです。  
ロール名は変更してOKです。  

割り当てるポリシーは以下です。  

* AmazonECSTaskExecutionRolePolicy

[ecsTaskExecutionRole.json](ecsTaskExecutionRole.json) というファイルを作成します。(ファイル内容はリンク先を参照)  


IAM ロールをコマンドで作成する場合は以下を実行します。  

```
ROLENAME=ecsTaskExecutionRole

aws iam create-role --role-name ${ROLENAME} --assume-role-policy-document file://ecsTaskExecutionRole.json
aws iam attach-role-policy --role-name ${ROLENAME} --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```


### ecsTaskCloud9test
ECS タスクが使用する IAM ロールです。  

ポリシーは無くても大丈夫です。(演習では)  
実際には ECS タスクから DynamoDB や S3 へアクセスしたい場合はここにポリシーを設定します。  

[ecsTaskCloud9test.json](ecsTaskCloud9test.json) というファイルを作成します。(ファイル内容はリンク先を参照)  

IAM ロールをコマンドで作成する場合は以下を実行します。  

```
ROLENAME=ecsTaskCloud9test

aws iam create-role --role-name ${ROLENAME} --assume-role-policy-document file://ecsTaskCloud9test.json
```

### ecsCodeDeployRole
CodeDeploy が使用するロールです。  

割り当てるポリシーは以下です。  

 * AWSCodeDeployRoleForECS

[ecsCodeDeployRole.json](ecsCodeDeployRole.json) というファイルを作成します。(ファイル内容はリンク先を参照)

IAM ロールをコマンドで作成する場合は以下を実行します。  

```
ROLENAME=ecsCodeDeployRole

aws iam create-role --role-name ${ROLENAME} --assume-role-policy-document file://ecsCodeDeployRole.json
aws iam attach-role-policy --role-name ${ROLENAME} --policy-arn arn:aws:iam::aws:policy/AWSCodeDeployRoleForECS
```

### AWSCodePipelineServiceRole-ap-northeast-1-bluegreen
CodePipeline が使用するロールです。  

後ほど使うので jq をインストールしておきます。  

```
sudo yum install -y jq
```

[AssumeCodepipeline.json](AssumeCodepipeline.json) というファイルを作成します。(ファイル内容はリンク先を参照)

[codepipelineservicerole.json](codepipelineservicerole.json) というファイルを作成します。(ファイル内容はリンク先を参照)

IAM ロールをコマンドで作成する場合は以下を実行します。  


```
ROLENAME=AWSCodePipelineServiceRole-ap-northeast-1-bluegreen

aws iam create-role --role-name ${ROLENAME} --path /service-role/ --assume-role-policy-document file://AssumeCodepipeline.json
POLICYARN=`aws iam create-policy --policy-name ${ROLENAME} --policy-document file://codepipelineservicerole.json | jq -r .Policy.Arn`
aws iam attach-role-policy --role-name ${ROLENAME} --policy-arn ${POLICYARN}
```

### cweCodepipelineRole
CloudWatch Events から CodePipeline を呼び出すためのロールを作成します。  

[cweCodepipelineRole.json](cweCodepipelineRole.json) というファイルを作成します。(ファイル内容はリンク先を参照)

[inlineCweCodepipelineRole.json](inlineCweCodepipelineRole.json) というファイルを作成します。(ファイル内容はリンク先を参照)

IAM ロールをコマンドで作成する場合は以下を実行します。  

```
ROLENAME=cweCodepipelineRole

aws iam create-role --role-name ${ROLENAME} --path /service-role/ --assume-role-policy-document file://cweCodepipelineRole.json
POLICYARN=`aws iam create-policy --policy-name ${ROLENAME} --policy-document file://inlineCweCodepipelineRole.json | jq -r .Policy.Arn`
aws iam attach-role-policy --role-name ${ROLENAME} --policy-arn ${POLICYARN}
```


----

<< [Step1](../step1/README.md)  |  [Step3](../step3/README.md)  >>
