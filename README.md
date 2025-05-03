# Project Zomboid Server on AWS

[English](#english) | [日本語](#japanese)

<a id="english"></a>
## Project Zomboid Dedicated Server on AWS

This CloudFormation template deploys a Project Zomboid dedicated server on AWS using ECS Fargate Spot instances with EFS persistent storage.

### Features

- **Cost-effective**: Uses Fargate Spot to reduce costs by up to 70% compared to regular Fargate
- **Dynamic infrastructure**: Automatically creates/deletes the Network Load Balancer when starting/stopping the server to minimize costs
- **Persistent storage**: EFS ensures your game world data persists between server restarts
- **Easy configuration**: Server settings stored in S3, easily updateable
- **Simple operation**: Start/stop server by adjusting a single parameter
- **No SSH required**: Fully managed solution that doesn't require direct server access

### Why I Developed This Solution

I developed this project to provide a simple, cost-effective way to run a Project Zomboid server with friends without the hassle of managing a dedicated server or dealing with SSH. Key benefits:

- **Low maintenance**: No need to SSH into the server for updates or management
- **Cost optimization**: Only pay for what you use - server resources when playing, minimal costs when idle
- **Easy to use**: Simple CloudFormation parameters for controlling the server

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
   - **Note:** Server startup takes approximately 10 minutes as it needs to:
     - Pull the Docker image
     - Download the latest Project Zomboid server files using SteamCMD
     - Configure and start the server

### Connecting to the Server

- Get the server address from the CloudFormation stack outputs (`GameNlbDnsName`)
- **Important:** This DNS name changes each time you restart the server (stop and start) since the NLB is recreated
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
- **Note:** This will automatically terminate the ECS task and delete the Network Load Balancer to save costs

### Cost Optimization

This solution implements several cost-saving measures:

1. **Fargate Spot** instances offer up to 70% discount over regular Fargate pricing
2. **Dynamic NLB provisioning** - The Network Load Balancer (which has an hourly cost) is only created when the server is running
3. **Zero-cost when stopped** - When the server is stopped, you only pay for the EFS storage containing your game data

### Required Ports

Project Zomboid uses several ports for different functions:

- **UDP 16261**: Main game connection port
- **TCP 16261-16293**: Game data and status information
- **UDP/TCP 8766-8767**: Steam query ports for server browser
- **TCP 27015**: RCON port
- **UDP 27015, 27031-27036**: Additional Steam communication ports

All these ports are automatically configured in the CloudFormation template.

### Troubleshooting

- **Server doesn't start**: Check the ECS task logs in CloudWatch
   - Logs are automatically collected in CloudWatch under `/ecs/[StackName]/zomboid`
   - You can view logs in AWS Console: CloudWatch > Log groups > `/ecs/[StackName]/zomboid`
- **Can't connect**: Verify you're using the correct and current `GameNlbDnsName` from the stack outputs
- **Configuration not applied**: Make sure your config files are in the correct S3 location
- **Slow server startup**: The initial startup takes approximately 10 minutes as the server needs to download and configure the game files

---

<a id="japanese"></a>
## AWS上のProject Zomboid専用サーバー

このCloudFormationテンプレートは、ECS Fargate SpotインスタンスとEFS永続ストレージを使用してAWS上にProject Zomboid専用サーバーをデプロイします。

### 特徴

- **コスト効率**: Fargate Spotを使用して通常のFargateと比較して最大70%のコスト削減
- **動的インフラストラクチャ**: サーバーの起動/停止時にNetwork Load Balancerを自動的に作成/削除してコストを最小化
- **永続ストレージ**: EFSによりサーバー再起動間もゲームワールドデータが保持される
- **簡単な構成**: サーバー設定はS3に保存され、簡単に更新可能
- **シンプルな操作**: 1つのパラメータを調整するだけでサーバーの起動/停止が可能
- **SSH不要**: サーバーに直接アクセスする必要のない完全管理型ソリューション

### このソリューションを開発した理由

私は専用サーバーの管理やSSHの扱いに悩まされることなく、友人とProject Zomboidサーバーを簡単かつコスト効率よく運用するためにこのプロジェクトを開発しました。主なメリット：

- **低メンテナンス**: 更新や管理のためにサーバーにSSHする必要なし
- **コスト最適化**: 使用した分だけ支払い - プレイ中はサーバーリソース、アイドル時は最小限のコスト
- **使いやすさ**: サーバーを制御するためのシンプルなCloudFormationパラメータ

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
   - **注意:** サーバーの起動には約10分かかります。これは以下の処理が必要なためです：
     - Dockerイメージのプル
     - SteamCMDを使用した最新のProject Zomboidサーバーファイルのダウンロード
     - サーバーの設定と起動

### サーバーへの接続

- CloudFormationスタックの出力から、サーバーアドレス（`GameNlbDnsName`）を取得
- **重要:** このDNS名はサーバーの再起動（停止と起動）のたびに変更されます。これはNLBが毎回再作成されるためです
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
- **注意:** これによりECSタスクが自動的に終了し、コスト削減のためにNetwork Load Balancerが削除されます

### コスト最適化

このソリューションはいくつかのコスト削減対策を実装しています：

1. **Fargate Spot**インスタンスは通常のFargate料金と比較して最大70%の割引を提供
2. **動的NLBプロビジョニング** - Network Load Balancer（時間単位のコストがかかる）はサーバー稼働時のみ作成
3. **停止時のゼロコスト** - サーバー停止時はゲームデータを含むEFSストレージのみ課金

### 必要なポート

Project Zomboidは様々な機能のために複数のポートを使用します：

- **UDP 16261**: メインゲーム接続ポート
- **TCP 16261-16293**: ゲームデータとステータス情報
- **UDP/TCP 8766-8767**: サーバーブラウザ用のSteamクエリポート
- **TCP 27015**: RCONポート
- **UDP 27015, 27031-27036**: Steamの追加通信ポート

これらのポートはすべてCloudFormationテンプレートで自動的に設定されます。

### トラブルシューティング

- **サーバーが起動しない**: CloudWatchでECSタスクログを確認
   - ログは自動的にCloudWatchの `/ecs/[スタック名]/zomboid` に収集されます
   - AWSコンソールで確認可能: CloudWatch > ロググループ > `/ecs/[スタック名]/zomboid`
- **接続できない**: スタック出力から正しい最新の`GameNlbDnsName`を使用していることを確認
- **設定が適用されない**: 設定ファイルが正しいS3の場所にあることを確認
- **サーバーの起動が遅い**: 初回起動は約10分かかります。サーバーがゲームファイルをダウンロードして設定する必要があるためです

---
