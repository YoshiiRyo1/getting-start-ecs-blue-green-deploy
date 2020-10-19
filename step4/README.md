# Step 4
## ALB
ALB を作成します。  
Cloud9 のターミナルを使用してコマンドを実行します。  


環境変数を設定します。  
ご自身の AWS 環境に合わせて編集してから実行します。  

```
export AWS_DEFAULT_REGION=ap-northeast-1

LBNAME=my-load-balancer                 # ALB 名
TG01NAME=${LBNAME}-target01             # Target Group 1
TG02NAME=${LBNAME}-target02             # Target Group 2

VPCID=vpc-xxxxxxxxxxxxxxxxx             # ALB が使う VPC ID
SUBNET1=subnet-xxxxxxxxxxxxxxxxx        # ALB が使うパブリックサブネット
SUBNET2=subnet-yyyyyyyyyyyyyyyyy        # ALB が使うパブリックサブネット
LBSECURITYGROUP=sg-xxxxxxxxxxxxxxxxx    # ALB が使うセキュリティグループ
```

ALB を作成するコマンドです。  
ALB を1つ、ターゲットグループを2つ、ヘルスチェックパスを「/health」にしています。  
※ 環境変数を設定した同じターミナルで実行してください。  

```
aws elbv2 create-load-balancer --name ${LBNAME}  \
  --subnets ${SUBNET1} ${SUBNET2}  \
  --security-groups ${LBSECURITYGROUP}

aws elbv2 create-target-group --name ${TG01NAME} \
  --protocol HTTP \
  --port 80 \
  --vpc-id ${VPCID} \
  --health-check-protocol HTTP \
  --health-check-path "/health" \
  --target-type ip
  
aws elbv2 create-target-group --name ${TG02NAME} \
  --protocol HTTP \
  --port 80 \
  --vpc-id ${VPCID} \
  --health-check-protocol HTTP \
  --health-check-path "/health" \
  --target-type ip

LBARN=`aws elbv2 describe-load-balancers --name ${LBNAME} | jq -r .LoadBalancers[].LoadBalancerArn`
TG01ARN=`aws elbv2 describe-target-groups --names ${TG01NAME} | jq -r .TargetGroups[].TargetGroupArn`

aws elbv2 create-listener --load-balancer-arn ${LBARN} \
  --protocol HTTP --port 80  \
  --default-actions Type=forward,TargetGroupArn=${TG01ARN}
```


## ECS クラスターの作成
以下のコマンドで ECS クラスタを作成します。  

```
aws ecs create-cluster --cluster-name bluegreen-cluster 
```

## タスク定義
ECS でコンテナを起動するための設定をします。  
以下の内容で [bluegreen-task.json](bluegreen-task.json) というファイルを作成します。  
(aws_account_id はご自身のものへ変更)   

以下のコマンドでタスク定義を登録します。  

```
aws ecs register-task-definition --cli-input-json file://bluegreen-task.json 
```

## ECS サービス作成
ECS サービスを作成します。これによってコンテナが起動します。  
環境変数は冗長なので、全ての手順を同じターミナルで実行しているのであれば省略して大丈夫です。  

```
export AWS_DEFAULT_REGION=ap-northeast-1

VPCID=vpc-xxxxxxxxxxxxxxxxx             # ECS クラスターが使う VPC ID
SUBNET1=subnet-xxxxxxxxxxxxxxxxx        # ECS クラスターが使うパブリックサブネット
SUBNET2=subnet-yyyyyyyyyyyyyyyyy        # ECS クラスターが使うパブリックサブネット
ECSSECURITYGROUP=sg-xxxxxxxxxxxxxxxxx    # ECS クラスターが使うセキュリティグループ
TG01ARN=`aws elbv2 describe-target-groups --names ${TG01NAME} | jq -r .TargetGroups[].TargetGroupArn`

aws ecs create-service --cluster bluegreen-cluster \
  --service-name bluegreen-service \
  --desired-count 1 \
  --task-definition bluegreen:1 \
  --load-balancers targetGroupArn=${TG01ARN},containerName=nginx1192,containerPort=80 \
  --launch-type FARGATE \
  --platform-version 1.4.0 \
  --network-configuration "awsvpcConfiguration={subnets=[${SUBNET1},${SUBNET2}],securityGroups=[${ECSSECURITYGROUP}],assignPublicIp=ENABLED}" \
  --deployment-controller type=CODE_DEPLOY
```

### 起動確認
数分待つと起動すると思います。  
以下のコマンドを実行し、「"(service bluegreen-service) has reached a steady state."」の出力が確認できれば OK です。  

```
aws ecs describe-services --cluster bluegreen-cluster --services bluegreen-service 
～～省略～～
            "events": [
                {
                    "id": "3574426a-8f22-4d6d-b53d-a0398e4d6c5b",
                    "createdAt": 1602552754.464,
                    "message": "(service bluegreen-service) has reached a steady state."
                },
～～省略～～
```

### ブラウザで確認
ALB の FQDN でブラウザからアクセスし、「HTML Sample」が表示されることを確認します。  
(セキュリティグループはよしなに変更してください。)  

----

[Step5](../step5/README.md)
