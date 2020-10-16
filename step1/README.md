# Step 1
## IAM ロール その1
IAM ロールを作成します。  

### EC2Cloud9Role
EC2 インスタンスプロファイルです。Cloud9 のインスタンスで使用します。   
ロール名は EC2Cloud9Role としました。(任意に変更してOK)  

マネジメントコンソール <a href="https://console.aws.amazon.com/iam/home?region=ap-northeast-1#/roles" target="_blank">IAM ロール</a> を開きます。  

**ロールの作成** をクリックします。  

**ユースケースの選択** → **一般的なユースケース** → **EC2** を選択して、**次のステップ** へ進みます。  

Attach アクセス権限ポリシー画面で割り当てるポリシーは以下です。  

 * AWSCodeCommitFullAccess
 * AmazonEC2ContainerRegistryFullAccess
 * ElasticLoadBalancingFullAccess
 * AmazonECS_FullAccess
 * IAMFullAccess
 * AWSCodePipeline_FullAccess 

このあとはロール名を入力してロールを作成します。  

## Cloud9
> AWS Cloud9 は、ブラウザのみでコードを記述、実行、デバッグできるクラウドベースの統合開発環境 (IDE) です。これには、コードエディタ、デバッガー、ターミナルが含まれています。Cloud9 には、JavaScript、Python、PHP などの一般的なプログラム言語に不可欠なツールがあらかじめパッケージ化されているため、新しいプロジェクトを開始するためにファイルをインストールしたり、開発マシンを設定したりする必要はありません。Cloud9 IDE はクラウドベースのため、インターネットに接続されたマシンを使用して、オフィス、自宅、その他どこからでもプロジェクトに取り組むことができます。また、Cloud9 では、サーバーレスアプリケーションを開発するためのシームレスなエクスペリエンスが提供されており、リソースの定義、デバッグ、ローカルとリモートの間でのサーバーレスアプリケーションの実行の切り替えを簡単に行えます。Cloud9 を使用すると、開発環境をすばやくチームと共有し、ペアプログラミングを行って互いの入力をリアルタイムで追跡できます。

*https://aws.amazon.com/jp/cloud9/ より引用*

### Cloud9 インスタンスの作成
<a href="https://ap-northeast-1.console.aws.amazon.com/cloud9/home" target="_blank">Cloud9 画面</a> を開きます。  

**Create environment** をクリックします。  

任意の名称を入力して、次へ進みます。  

Environment Type は **Create a new EC2 instance for environment (direct access)** を選択します。  

**Network Setting** を展開して、前の手順で作成した VPC とパブリックサブネットを選択します。  

### インスタンスプロファイルの作成
<a href="https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#Instances:" target="_blank">EC2 インスタンス画面</a> を開くと **aws-cloud9-xxxx** というインスタンスが起動しているはずです。  
そのインスタンスに前の手順で作成したインスタンスプロファイルを関連付けます。    

### AMTC 無効
AWS Managed Temporary Credentials (AMTC) を無効にしておきます。  
無効にする方法は以下に詳しいです。  

[Cloud9からIAM Roleの権限でAWS CLIを実行する](https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/execute-aws-cli-with-iam-role-on-cloud9/)


### Cloud9 IDE を開く
Cloud9 画面に戻り **Open IDE** をクリックします。  
しばらくすると IDE が開きます。  

### docker-compose
今回はイメージ作成のために docker-compose を使用します。  

Cloud9 画面下部にコンソールがあるはずです。  

まずは docker がインストールされていることの確認をします。  
```
docker -v
```

万が一インストールされていなければインストールします。  
```
sudo yum install -y docker
```

次に docker-compose をインストールします。  

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

docker-compose -v
```

----

[Step2](../step2/README.md)
