# flask アプリ

## コマンド

```sh
# ローカルサーバ起動
make run
```

## Azure App Service へのデプロイ

### コマンド

```sh
# Azure にログイン
# ※ az コマンドがない場合、 brew install azure-cli でインストール
az login

##### Azure App Service にデプロイ #####

# B1: ¥8,266.703/month
# ※ sku: Stock Keeping Unit
az webapp up --runtime PYTHON:3.12 --sku B1 --logs

# F1: ¥0
az webapp up --runtime PYTHON:3.12 --sku F1 --logs

# F1 + 各種指定(アプリ名, リソースグループ名, リージョン)
# ※ AppServicePlan は事前に作成していない想定
az webapp up --name $WEBAPP --resource-group $RESOURCE_GROUP --runtime PYTHON:3.12 --sku F1 --location japaneast --logs


※ アプリ名はグローバルでユニークである必要がある
※ 一般的に大文字は使用せず、小文字、数字、ハイフン、アンダースコアを組み合わせる
※ App Service Plan は自動で作成され、Azure 側で命名される
  => 名前を指定したいなら事前に作成しておく必要がある
  [作成順序]
  リソースグループ => AppServicePlan => webapp => ソースコードが入った zip
※ App Service Plan が定義するもの
  - オペレーティング システム (Windows、Linux)
  - リージョン (米国西部、米国東部など)
  - VM インスタンスの数
  - VM インスタンスのサイズ (小、中、大)
  - 価格レベル (Free、Shared、Basic、Standard、Premium、PremiumV2、PremiumV3、Isolated、IsolatedV2)
    ※ デフォルトでは無料の F1 プランが設定される

# リソースのクリーンアップ
az group delete --name $RESOURCE_GROUP --no-wait

##### App Service Plan の作成 #####
# リソースグループの作成
# ※ App Service Plan の作成にはリソースグループが必要
az group create --name $RESOURCE_GROUP --location $LOCATION

# App Service Plan の作成
az appservice plan create --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --sku F1 --is-linux


# 作成した App Service Plan の確認
az appservice plan list --resource-group $RESOURCE_GROUP --output table


# webapp の作成
az webapp up --name $WEBAPP_PLAN --resource-group $RESOURCE_GROUP --plan  --runtime PYTHON:3.12 --sku F1 --location $LOCATION --logs


##### その他 #####
# 使用可能な全てのランタイムを一覧表示
az webapp list-runtimes --os linux --output table

```

## IaC

- 該当ファイル: ias.json

```sh
# テンプレートをもとにリソースをデプロイ
az deployment group create --resource-group $RESOURCE_GROUP --template-file template.json --parameters @parameters.json

# VM で SSH を有効にする
az vm extension set --resource-group $RESOURCE_GROUP --vm-name $VIRTUAL_MACHINE --name WindowsOpenSSH --publisher Microsoft.Azure.OpenSSH --version 3.0

# VM に接続
az ssh vm --local-user $VIRTUAL_MACHINE_USER --resource-group $RESOURCE_GROUP --name $VIRTUAL_MACHINE

# TCP 22 port を開く
az network nsg rule create -g $RESOURCE_GROUP --nsg-name $NETWORK_SECURITY_GROUP -n allow-SSH --priority 1000 --source-address-prefixes 208.130.28.4/32 --destination-port-ranges 22 --protocol TCP
```

- template.json の書き方参照元
  - Microsoft Azure 実践ガイド (書籍)
  - [[公式] Resource Manager テンプレートから Windows 仮想マシンを作成する](https://learn.microsoft.com/ja-jp/azure/virtual-machines/windows/ps-template)<br>
    JSON 形式でリソース構築用テンプレートを書く方法を記載

## 参考資料

- [[公式] App Service の概要](https://learn.microsoft.com/ja-jp/azure/app-service/overview)
- [[公式] クイック スタート: Python (Django、Flask、または FastAPI) Web アプリを Azure App Service にデプロイする](https://learn.microsoft.com/ja-jp/azure/app-service/quickstart-python?tabs=flask%2Cwindows%2Cazure-cli%2Cazure-cli-deploy%2Cdeploy-instructions-azportal%2Cterminal-bash%2Cdeploy-instructions-zip-azcli)
- [[公式]名前付け規則を定義する](名前付け規則を定義する)
  ```sh
  # Azure 推奨命名規則
  {リソースの種類}-{プロジェクト名}-{env}-{region}-{番号/ID/カウンター}
  # 例 (リソースグループ)
  rg-gettingstarteddemo-linuxvm-eus2-dev-01
  ※ eus2: East US 2 (Virginia, United States)
  ※ gettingstarteddemo-linuxvm がプロジェクト名
  ```
- [[公式] Azure regions list](https://learn.microsoft.com/ja-jp/azure/reliability/regions-list)
  ```sh
  # 日本
  Japan East (Tokyo, Saitama): jpn
  Japan West (Osaka): jpnw
  ```
- [[公式] Azure リソースの省略形の例](https://learn.microsoft.com/ja-jp/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations)
  Virtual machines: vm<br>
  AppServicePlan: asp<br>
  etc...
- [[公式] Secure Shell (SSH) を使用して接続し、Windows が動作している Azure 仮想マシンにサインオンする方法](https://learn.microsoft.com/ja-jp/azure/virtual-machines/windows/connect-ssh?tabs=azurecli)<br>
  Windows の VM にリモートから SSH 接続するまでの方法を記載
