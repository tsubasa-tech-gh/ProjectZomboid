# Project Zomboid Server on AWS

[English](#english) | [日本語](#japanese)

<a id="english"></a>
## Project Zomboid Dedicated Server on AWS

This CloudFormation template deploys a Project Zomboid dedicated server on AWS using ECS Fargate Spot instances with EFS persistent storage.

### Features

- **Cost-effective**: Uses Fargate Spot to reduce costs
- **Persistent storage**: EFS ensures your game world data persists between server restarts
- **Easy configuration**: Server settings stored in S3, easily updateable
- **Simple operation**: Start/stop server by adjusting a single parameter

### Prerequisites

- AWS account
- Basic knowledge of AWS CloudFormation
- Project Zomboid game for extracting server configuration files

### Setup Instructions

1. **Deploy the CloudFormation Stack**
   - Log in to AWS Management Console
   - Go to CloudFormation
   - Create stack > With new resources
   - Upload `project-zomboid.yaml`
   - Fill in the required parameters (server name, admin password, etc.)
   - **Important:** Keep `DesiredTaskCount` set to 0 for initial deployment
   - Create stack

2. **Upload Configuration Files**
   - Extract configuration files from your Windows Project Zomboid installation
     - Navigate to `C:\Users\<YourUser>\Zomboid\Server`
     - Copy the following files:
       - `servertest.ini` (or create new one)
       - `servertest_SandboxVars.lua`
       - `servertest_spawnpoints.lua`
       - `servertest_spawnregions.lua`
   - Upload these files to the S3 bucket created by CloudFormation
     - The bucket name is shown in the stack's outputs
     - Upload files to the `pz-config/` folder (create this folder if it doesn't exist)
     - Keep the original filenames (`servertest.ini`, etc.)

3. **Start the Server**
   - Go to CloudFormation > Stacks > Your stack > Update
   - Change `DesiredTaskCount` from 0 to 1
   - Update stack

### Connecting to the Server

- Get the server address from the CloudFormation stack outputs (`GameNlbDnsName`)
- Use the admin password you specified in the CloudFormation parameters
- In Project Zomboid, add a server with this address
- Default port: 16261/UDP

**Detailed connection steps:**
1. Launch Project Zomboid
2. Select "Join" from the main menu
3. Click "Add Server"
4. Enter a name for the server (any name you want)
5. Enter the `GameNlbDnsName` address from CloudFormation outputs
6. Ensure port is set to 16261
7. Click "Save"
8. Select your server from the list and click "Join"
9. Enter your username and the admin password you set in CloudFormation

### Stopping the Server

- Go to CloudFormation > Stacks > Your stack > Update
- Change `DesiredTaskCount` from 1 to 0
- Update stack

### Troubleshooting

- **Server doesn't start**: Check the ECS task logs in CloudWatch
   - Logs are automatically collected in CloudWatch under `/ecs/[StackName]/zomboid`
   - You can view logs in AWS Console: CloudWatch > Log groups > `/ecs/[StackName]/zomboid`
- **Can't connect**: Verify your security group allows the required ports (UDP 16261-16272, TCP 16262-16272)
- **Configuration not applied**: Make sure your config files are in the correct S3 location

---

<a id="japanese"></a>
## AWS上のProject Zomboid専用サーバー

このCloudFormationテンプレートは、ECS Fargate SpotインスタンスとEFS永続ストレージを使用してAWS上にProject Zomboid専用サーバーをデプロイします。

### 特徴

- **コスト効率**: Fargate Spotを使用してコストを削減
- **永続ストレージ**: EFSによりサーバー再起動間もゲームワールドデータが保持される
- **簡単な構成**: サーバー設定はS3に保存され、簡単に更新可能
- **シンプルな操作**: 1つのパラメータを調整するだけでサーバーの起動/停止が可能

### 前提条件

- AWSアカウント
- AWS CloudFormationの基本知識
- サーバー設定ファイル抽出用のProject Zomboidゲーム

### セットアップ手順

1. **CloudFormationスタックをデプロイする**
   - AWSマネジメントコンソールにログイン
   - CloudFormationに移動
   - スタックの作成 > 新しいリソースを使用
   - `project-zomboid.yaml`をアップロード
   - 必要なパラメータ（サーバー名、管理者パスワードなど）を入力
   - **重要:** 初回デプロイ時は`DesiredTaskCount`を0のままにする
   - スタックを作成

2. **設定ファイルをアップロードする**
   - WindowsのProject Zomboidインストールから設定ファイルを抽出
     - `C:\Users\<ユーザー名>\Zomboid\Server`に移動
     - 以下のファイルをコピー:
       - `servertest.ini` (または新規作成)
       - `servertest_SandboxVars.lua`
       - `servertest_spawnpoints.lua`
       - `servertest_spawnregions.lua`
   - これらのファイルをCloudFormationで作成したS3バケットにアップロード
     - バケット名はスタックの出力に表示されます
     - `pz-config/`フォルダにファイルをアップロード (フォルダが存在しない場合は作成)
     - 元のファイル名を維持 (`servertest.ini`など)

3. **サーバーを起動する**
   - CloudFormation > スタック > あなたのスタック > 更新に移動
   - `DesiredTaskCount`を0から1に変更
   - スタックを更新

### サーバーへの接続

- CloudFormationスタックの出力から、サーバーアドレス（`GameNlbDnsName`）を取得
- CloudFormationパラメータで指定した管理者パスワードを使用
- Project Zomboid内で、このアドレスでサーバーを追加
- デフォルトポート: 16261/UDP

**詳細な接続手順:**
1. Project Zomboidを起動
2. メインメニューから「参加」を選択
3. 「サーバー追加」をクリック
4. サーバーの名前を入力（任意の名前）
5. CloudFormation出力の`GameNlbDnsName`アドレスを入力
6. ポートが16261に設定されていることを確認
7. 「保存」をクリック
8. リストからサーバーを選択し、「参加」をクリック
9. ユーザー名とCloudFormationで設定した管理者パスワードを入力

### サーバーの停止

- CloudFormation > スタック > あなたのスタック > 更新に移動
- `DesiredTaskCount`を1から0に変更
- スタックを更新

### トラブルシューティング

- **サーバーが起動しない**: CloudWatchでECSタスクログを確認
   - ログは自動的にCloudWatchの `/ecs/[スタック名]/zomboid` に収集されます
   - AWSコンソールで確認可能: CloudWatch > ロググループ > `/ecs/[スタック名]/zomboid`
- **接続できない**: セキュリティグループが必要なポート（UDP 16261-16272、TCP 16262-16272）を許可していることを確認
- **設定が適用されない**: 設定ファイルが正しいS3の場所にあることを確認

---
